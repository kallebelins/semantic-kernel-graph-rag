# Consulta local k-hop com evidências citadas

## Contexto e Problema
Quando a pergunta do usuário envolve fatos específicos sobre entidades (pessoas, organizações, sistemas, normas) e relações diretas entre elas, a melhor estratégia é uma expansão local no grafo em poucos saltos (k-hop). Precisamos coletar evidências ancoradas em `TextUnit` próximos às entidades e relações relevantes e sintetizar uma resposta curta com citações verificáveis.

## Objetivo
Responder perguntas objetivas com base no subgrafo do projeto, navegando 1–3 saltos a partir de entidades sementes detectadas na pergunta, coletando `TextUnit` de evidência via arestas `MENTIONED_IN`/`SUPPORTED_BY` e retornando uma resposta curta (≤8 linhas) com citações `{doc_id, unit_id, page/section/heading}`.

## Atores e Escopo
- Usuários: Analista, Operador, Usuário de Negócio
- Serviços: `LocalGraph` (SKGraph), `IGraphStore` (Neo4j), `ILexicalIndex`/`IVectorIndex` (opcional para re-rank), Observabilidade
- Escopo: Apenas consulta local. Não cobre detecção de comunidades nem QFS global.

## Pré-condições
- Grafo consolidado do projeto atualizado: entidades, relações e claims com evidências (`unit_id`) já mesclados
- Índices lexical/vetorial disponíveis para reclassificação de evidências (opcional)
- Perfis de prompt (`answering`) e políticas de sensibilidade carregados
- Constraints únicas ativas (ver `docs/roadmap_tecnico.md` §3.2)

## Entradas
- Pergunta: `question: string`
- Filtros: `filters {tags[], setor, tema, lang}`
- Parâmetros: `params {k, depth, beam_width, rerank, max_tokens_ctx}`
- Contexto: `project_id`, `user_id` (para RBAC/ABAC)

## Fluxo (alto nível)
- Detectar entidades sementes na pergunta (EL simples ou BM25/ANN em nomes/aliases de `Entity`)
- Expandir k-hop (1..depth) com feixe controlado (`beam_width`) a partir das sementes
- Coletar evidências:
  - `Entity` → `MENTIONED_IN` → `TextUnit`
  - `Relation/Claim` → `SUPPORTED_BY` → `TextUnit`
- Selecionar top evidences (BM25 ∪ ANN) e aplicar re-rank (opcional)
- Síntese com `PromptProfile.answering`, respeitando `max_tokens_ctx` e citando evidências
- Produzir trace XAI: nós/arestas visitados, métricas de custo/latência
- Nós SKGraph: `LocalGraph` (+ `RetryPolicyGraphNode`, `ResultAggregator` quando fan-out)

## Regras/Políticas e Guardrails
- Citações obrigatórias: toda resposta deve conter ao menos 2 citações válidas `{doc_id, unit_id}`
- Filtros de governança: aplicar RBAC/ABAC e sensibilidade por `TextUnit` antes da exposição
- Orçamento: limitar `depth ≤ 3`, `beam_width ≤ 10`, `max_tokens_ctx` conforme projeto
- Anti-explosão: penalizar hubs (alto grau) no feixe; priorizar relações com evidência recente/relevante

## Saídas e Artefatos
- `answer: string` (≤8 linhas)
- `citations[]`: `{doc_id, unit_id, page?, section?, heading?, lang}`
- `trace`: `{nodes[], edges[], steps[], metrics{latency_ms, cost.tokens_in/out, provider/model}}`
- `used_params`: parâmetros efetivos após clamps de orçamento

## Parâmetros e Ajustes
- `depth` (1–3): número de saltos na expansão; maior depth aumenta recall e custo
- `beam_width` (1–10): número de caminhos mantidos por passo; controla explosão
- `k` (5–50): quantidade-alvo de evidências candidatas antes de re-rank
- `rerank` (bool|topN): aplica reclassificador cross-encoder nas evidências
- `max_tokens_ctx` (int): teto de tokens de contexto para a síntese

## Métricas e Critérios de Aceite
- Groundedness: 100% de citações com `{doc_id, unit_id}` válidos
- Latência p95 dentro do orçamento do projeto para `depth≤2, beam≤5, k≤20`
- Qualidade: relevância ≥ alvo interno, cobertura das entidades principais da pergunta
- Observabilidade: trace completo com nós/arestas visitados e custos

## Rastreabilidade
- Roadmap: §5 Modos de Consulta (Local GraphRAG), §11 Operação e Custos
- Roadmap técnico: §2.2 Consulta (LocalGraph), §3.2 Grafo (Neo4j), §18.3 I/O (LocalGraph)

## Riscos e Considerações
- Linkagem de entidades ambígua → usar aliases e filtro por `lang/tags/setor/tema`
- Explosão por hubs → limitar grau e aplicar heurística por tipo de relação
- Evidências antigas ou fracas → re-rank e thresholds mínimos
- Conteúdo sensível → redigir/mascarar conforme política antes da montagem do contexto

## Exemplos
Requisição de API:
```json
{
  "project_id": "p-001",
  "question": "Qual a relação entre Contoso e Fabrikam em 2024?",
  "mode": "local_graph",
  "filters": {"lang": "pt", "setor": "Tecnologia"},
  "params": {"depth": 2, "beam_width": 5, "k": 20, "rerank": true, "max_tokens_ctx": 2000}
}
```

Resposta (resumo):
```json
{
  "answer": "Em 2024, Contoso e Fabrikam estabeleceram parceria para iniciativas de IA responsável.",
  "citations": [
    {"doc_id": "d-42", "unit_id": "u-000123", "page": 3, "section": "Resultados", "heading": "Parcerias 2024"},
    {"doc_id": "d-42", "unit_id": "u-000145"}
  ],
  "trace": {"nodes": ["Entity:Contoso","Entity:Fabrikam","Relation:Partnership"], "edges": ["REL","SUPPORTED_BY"], "metrics": {"latency_ms": 820, "cost": {"tokens_in": 1450, "tokens_out": 200}}},
  "used_params": {"depth": 2, "beam_width": 5, "k": 20, "rerank": true, "max_tokens_ctx": 2000}
}
```

Expansão k-hop (conceitual em Cypher):
```cypher
// Sementes por nome/alias
MATCH (s:Entity {projectId:$projectId})
WHERE toLower(s.name) IN $seedNames OR any(a IN s.aliases WHERE toLower(a) IN $seedNames)
WITH collect(s) AS seeds

// Expansão até 2 saltos priorizando relações com evidência
MATCH p=(s IN seeds)-[r:REL|MENTIONED_IN|SUPPORTED_BY*1..2]-()
WHERE ALL(rel IN relationships(p) WHERE coalesce(rel.projectId, $projectId) = $projectId)
RETURN p LIMIT 500
```

## Variantes e Extensões
- Híbrido (DRIFT): usar pré-filtro global por comunidades antes da expansão local
- Multimodal: incluir evidências com `time_range` (áudio/vídeo) e `bbox` (imagem)
- Fallback: se poucas evidências, cair para `Vector RAG` com os mesmos filtros
