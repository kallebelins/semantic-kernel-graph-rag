# Usecase: Local GraphRAG (expansão por entidades)

## Contexto e Problema
Perguntas que envolvem entidades específicas (pessoas, sistemas, normas, processos) e relações diretas entre elas se beneficiam de uma busca no grafo com expansão limitada (k-hop). O Vector RAG pode não capturar estrutura relacional, e o Global QFS é excessivo. Precisamos navegar o subgrafo do projeto para reunir evidências estruturadas e citáveis.

## Objetivo
Responder perguntas factuais locais explorando o grafo (Neo4j) a partir de entidades relevantes, expandindo poucos saltos, coletando evidências em `TextUnit` e sintetizando resposta curta com citações e traço do subgrafo explorado.

## Atores e Escopo
- Usuário final (via `POST /v1/query`, `mode=local_graph`)
- Serviços: `IGraphStore` (Neo4j), `EntityResolver`, `SubgraphBuilder`, `EvidenceCollector`, `AnswerSynthesizer`, `Guardrails`
- Dentro: resolução de entidades, expansão k-hop, coleta de evidências, resposta citável, trace XAI local
- Fora: comunidades/relatórios globais (tratado em QFS) e DRIFT

## Pré-condições
- Grafo consolidado com `Entity`, `Relation`, `Claim`, `TextUnit` e arestas de evidência (`MENTIONED_IN`, `SUPPORTED_BY`)
- Config `PromptProfiles.answering` orientado a respostas curtas e citáveis
- Esquemas de IE normalizados e alias resolvidos para alta precisão na resolução inicial

## Entradas
- `{ project_id, question, mode=local_graph, filters {tags,setor,tema,lang}, params {k, depth, beam_width, max_tokens_ctx} }`
- Entidades candidatas extraídas da pergunta (resolver por nome/alias/tipo)

## Fluxo (alto nível)
- Etapa 1: Resolver entidades centrais da pergunta (NLP + lookup por alias)
- Etapa 2: Construir subgrafo inicial (nós/arestas incidentes) e definir fronteira
- Etapa 3: Expansão k-hop controlada (`depth`, `beam_width`) priorizando relações/claims com maior suporte
- Etapa 4: Coletar `TextUnit` de evidência ligados a nós/arestas relevantes
- Etapa 5: Síntese curta com citações e verificação de políticas (PII/sensibilidade)
- Etapa 6: Serializar traço (nós/arestas visitados, caminhos, evidências) e métricas
- Nós SKGraph: `LocalGraph` → `EntityResolver` → `SubgraphExpand` → `EvidenceCollect` → `AnswerSynthesizer` → `Guardrails`

## Regras/Políticas e Guardrails
- Limitar `depth<=3` e `beam_width` por orçamento
- Filtrar nós/arestas por `projectId` e taxonomias; excluir conteúdo confidencial por ABAC
- Priorizar caminhos com claims de maior confiança e maior densidade de evidências

## Saídas e Artefatos
- `answer` com citações `{doc_id, unit_id, page/section/heading}`
- `trace.subgraph` JSON: `{nodes[], edges[], paths[], metrics}`
- `metrics`: `{latency_ms, tokens_in/out, expansions, collected_units}`

## Parâmetros e Ajustes
- `depth` (2–3), `beam_width` (3–10), `k` alvo de `TextUnit` evidência (ex.: 20–40)
- `max_tokens_ctx` (ex.: 2k); ranking de evidências por `score = w1*support + w2*centrality + w3*lexical_match`

## Métricas e Critérios de Aceite
- Precisão em perguntas relacionais locais superior ao Vector RAG isolado
- Groundedness ≥ 0.95; traço inclui caminhos usados na resposta
- Latência P95 ≤ 4s sob `depth<=3` e `beam_width<=8`

## Rastreabilidade
- Roadmap: seções 5 (Local GraphRAG), 3.2 (Grafo), 7/10 (Observabilidade)
- Roadmap técnico: seções 5 (Modos), 3.2 (Neo4j), 18 (Contratos I/O por nó)

## Riscos e Considerações
- Resolução de entidades ambígua: desambiguação por tipo/alias e feedback do usuário
- Explosão combinatória na expansão: controle estrito de `depth`/`beam_width` e heurísticas de corte
- Cobertura limitada se IE estiver subextraída: usar gleanings seletivos na ingestão

## Exemplos
- Pergunta: "Quais sistemas dependem do Sistema X?"
- Resposta cita caminhos `(:Entity)-[:REL {type:"depende"}]->(:Entity)` com `SUPPORTED_BY` → `TextUnit`
- `trace.subgraph` mostra os nós visitados e as evidências usadas

## Variantes e Extensões
- Inclui `Claim` com polaridade e confiança para distinguir afirmações controversas
- Integração com Visualização (Mermaid/Neo4j Bloom/GraphView) a partir do `trace.subgraph`
