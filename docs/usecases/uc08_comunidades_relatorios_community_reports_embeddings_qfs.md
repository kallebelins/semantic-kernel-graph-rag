# Usecase: Community Reports e embeddings para QFS

## Contexto e Problema
Após detectar comunidades hierárquicas, precisamos sintetizar conhecimento por comunidade em relatórios consistentes, com citações e estrutura padrão. Esses relatórios devem ser pesquisáveis globalmente via embeddings (QFS) e servir de base para panoramas e navegação macro.

## Objetivo
- Gerar `CommunityReport` por comunidade/nível com seções padronizadas (objetivos, riscos, processos, dependências, "3 coisas a saber", "3 lacunas"), citando `community_id` e evidências.
- Indexar embeddings de relatórios em `Vector Store` para suportar GraphRAG Global (QFS).

## Atores e Escopo
- Serviços: `ReportService` (sumarização), `CommunityService` (fornece membros/centroides), `EmbeddingService`, `IVectorIndex`, `GraphStore`.
- Escopo: sumarização e indexação vetorial dos relatórios; consulta global ocorre em UC11.

## Pré-condições
- Comunidades persistidas (UC07) com `(:Entity)-[:IN_COMMUNITY {level}]->(:Community)`.
- Perfis de prompt e limites de orçamento definidos (`PromptProfile.summarization`).
- `Vector Store` disponível e configurado por `tenant/project`.

## Entradas
- `project_id`, `communityIds` por nível e `PromptProfile` de sumarização.
- Parâmetros: `max_tokens_ctx`, `top_central_entities`, `max_evidence_per_section`, `lang`, `sensitivity_filters`.

## Fluxo (alto nível)
- Etapa 1: Selecionar comunidades alvo (por nível) e recuperar entidades centrais e unidades de evidência relevantes.
- Etapa 2: Montar contexto controlado por orçamento com trechos/evidências mais informativas.
- Etapa 3: Executar sumarização (`ReportService.SummarizeAsync`) gerando seções padronizadas e citações.
- Etapa 4: Persistir `CommunityReport` e vincular a `Community` (armazenamento + referência no grafo ou relacional).
- Etapa 5: Gerar embedding do relatório e indexar em `IVectorIndex` com payload `{projectId, communityId, level, lang, tags}`.
- Etapa 6: Registrar telemetria de custo/latência por relatório e por nível.
- Nós SKGraph (exemplos): `FunctionGraphNode(ListCommunities)`, `ReportSummarize(level)`, `Embedder(CommunityReport)`, `ResultAggregator(IndexReports)`.

## Regras/Políticas e Guardrails
- Citações obrigatórias: cada seção deve referenciar `community_id` e evidências agregadas (ex.: `topEvidenceUnitIds`).
- Orçamento: limitar `max_tokens_ctx` e número de evidências por seção; fallback para sumários mais curtos.
- Privacidade: aplicar `sensitivity_filters` e políticas (RBAC/ABAC) antes de compor o contexto e exibir saídas.
- Idempotência: chave `{projectId, communityId, reportVersionParams}` para cache e reuso.
- Idioma: produzir no idioma do projeto; manter consistência terminológica.

## Saídas e Artefatos
- `CommunityReport {id, communityId, level, text, sections{...}, citations{communityId, unitIds[]}, lang}`.
- Embedding do relatório e referência em `IVectorIndex` com payload para filtros.
- Métricas de geração: `cost.tokens_in/out`, `latency_ms`, `provider/model`.

## Parâmetros e Ajustes
- `top_central_entities` (ex.: 5–10) e `max_evidence_per_section` (ex.: 3–5).
- `max_tokens_ctx` para compor contexto e `max_tokens_out` por relatório.
- `embedder_model` e `dimension` do índice vetorial.
- `sections_enabled[]` para ativar/desativar seções conforme o caso de uso.

## Métricas e Critérios de Aceite
- Métricas: `latency_ms` por relatório, custo em tokens, cobertura de evidências, qualidade subjetiva (relevância/groundedness/diversidade).
- Critérios de aceite:
  - Relatórios gerados para ≥ 90% das comunidades alvo por nível com citações válidas.
  - Embeddings indexados com payload correto e pesquisáveis via QFS.
  - Respeito às políticas de sensibilidade e idioma configurados.

## Rastreabilidade
- Roadmap: `docs/roadmap.md` seções 8 (Comunidades e Relatórios) e 5 (Modos de Consulta/QFS).
- Roadmap técnico: `docs/roadmap_tecnico.md` seções 2.2/2.3 (Consulta Global/QFS e Comunidades), 3.2 (Modelo de Grafo), 12 (Prompts), 7 (Observabilidade).

## Riscos e Considerações
- Contexto insuficiente pode gerar sumários superficiais; mitigar com seleção dinâmica e gleanings pontuais.
- Custo elevado para muitos relatórios; usar paralelismo controlado e cache/idempotência.
- Desalinhamento de idioma; checar `lang` por projeto e ajustar perfis.
- Deriva semântica entre execuções; versionar `PromptProfile` e parâmetros de geração.

## Exemplos
Estrutura de saída (conceitual):
```json
{
  "id": "rep_c_42_l2",
  "communityId": "c_42",
  "level": 2,
  "lang": "pt",
  "sections": {
    "objetivos": "...",
    "riscos": ["..."],
    "processos": ["..."],
    "dependencias": ["..."],
    "top3_coisas": ["..."],
    "top3_lacunas": ["..."]
  },
  "citations": { "communityId": "c_42", "unitIds": ["u_10","u_77","u_130"] }
}
```

Documento vetorial (payload):
```json
{
  "id": "rep_c_42_l2",
  "vector": [0.01, 0.02, ...],
  "payload": {"projectId": "p1", "communityId": "c_42", "level": 2, "lang": "pt", "tags": ["seguranca","lgpd"]}
}
```

## Variantes e Extensões
- Cadeia hierárquica: sumarizar nível fino → compor sumários macro (nível superior) com map-reduce.
- Citações multimodais: incluir `time_range`, `bbox`, `page/section/heading` nas evidências (ver Multimodais e XAI).
- Atualização incremental: reprocessar apenas comunidades afetadas por alterações no grafo.
