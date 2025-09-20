# Padrão de saída Usecase: Playbooks de Insights (riscos, tendências, FAQ, matriz de conformidade)

## Contexto e Problema
Organizações precisam transformar o grafo de conhecimento e os artefatos de RAG em inteligência acionável para executivos e squads (riscos e lacunas, tendências mês a mês, FAQs recorrentes e aderência a requisitos regulatórios). Esses resultados são repetitivos, exigem citações auditáveis e precisam ser reproduzíveis, versionados por projeto/idioma e com custos controlados.

## Objetivo
Gerar insights curados e auditáveis a partir do grafo e dos índices (vetorial/BM25), com respostas explicáveis, citações obrigatórias e métricas de qualidade/custo. Entregas incluem relatórios de riscos e lacunas, panorama de tendências, FAQ generator e matriz de conformidade, integrados aos endpoints de Insights e prontos para exportação a BI.

## Atores e Escopo
- Usuários: Executivos, Gestores de Risco/Compliance, Analistas de Conhecimento, Owners de Projeto.
- Serviços: `InsightPlaybooks`, `GlobalQFS`, `LocalGraph`, `ReRanker`, `PIIFilter`, `CommunityDetection`, `ReportSummarizer`.
- Fora do escopo: Monitoração contínua de eventos (coberta em agente proativo); criação de dashboards BI (detalhada no usecase de exportações/BI).

## Pré-condições
- Projeto com ingestão e indexação concluídas (embeddings e BM25) e IE consolidada no grafo.
- Comunidades detectadas por nível e `CommunityReport` gerado com embeddings (QFS).
- Perfis de prompt e few-shots configurados para `summarization`, `answering` e `classification`.
- Políticas de segurança/privacidade definidas (RBAC/ABAC, sensibilidade) e chaves de acesso vigentes.

## Entradas
- `project_id`, `intent=Insights` e `type={risk|trend|faq|compliance_matrix}`.
- Filtros: `tags[]`, `setor`, `tema`, `lang`, `sensitivity<=threshold`.
- Parâmetros: `k`, `max_communities`, `beam_width`, `depth`, `rerank`, `date_range` (para tendências), `regulatory_set` (para conformidade), `top_n`.

## Fluxo (alto nível)
1) Selecionar playbook pelo `type` e preparar filtros/limiares.
2) Recuperar contexto:
   - Riscos/Lacunas: varrer comunidades e claims; identificar afirmações negativas/pendências; priorizar por centralidade/impacto.
   - Tendências: usar embeddings de `CommunityReport` e janelas temporais; comparar períodos (drift de vocabulário e tópicos).
   - FAQ: minerar perguntas comuns (histórico de `Query`) e conteúdo recorrente em relatórios/claims.
   - Conformidade: mapear `claims` → requisitos (`regulatory_set`) e lacunas.
3) Coletar evidências (TextUnits) por `LocalGraph` (k-hop) quando necessário para fundamentar cada insight.
4) Reclassificar e ordenar (`ReRanker`) por relevância, cobertura e groundedness.
5) Gerar saída estruturada e citável, com `trace` opcional do subgrafo/etapas.

- Nós SKGraph (exemplos): `SwitchGraphNode(Intent/Type)`, `GlobalQFS`, `LocalGraph`, `FunctionGraphNode(InsightPlaybooks.Generate)`, `ForeachLoopGraphNode(Communities)`, `ResultAggregator`, `RetryPolicyGraphNode`, `PIIFilter`.

## Regras/Políticas e Guardrails
- Aplicar RBAC/ABAC por `tenant/project/tag/setor/tema` e por sensibilidade de `TextUnit`.
- Citações obrigatórias por insight; mascarar/redigir PII conforme perfil do usuário.
- Orçamento por consulta (`max_tokens_ctx`, `max_steps`); timeouts/retries idempotentes por nó.

## Saídas e Artefatos
- Objeto `Insight`: `{ id, type, payload, score, linked_query_ids[], project_id, created_at }`.
- Payload por tipo:
  - `risk`: `{ risks[{title, description, severity, evidence[{doc_id, unit_id, page|time_range|bbox}], recommendations[]}] }`
  - `trend`: `{ periods[], topics[{name, delta, explanation, citations[]}] }`
  - `faq`: `{ qas[{question, answer, citations[]}] }`
  - `compliance_matrix`: `{ controls[], mapping[{control, claims[], gaps[], citations[]}] }`
- Persistência: `Insight` (SQL) e artefatos em `artifacts/insights/{project}/{type}/{timestamp}.json` (object storage).

## Parâmetros e Ajustes
- `k`, `rerank`, `max_tokens_ctx`, `max_communities`, `beam_width`, `depth`.
- `severity_threshold`, `gap_threshold`, `date_range`, `time_grain={month|quarter}`, `top_n`.
- `regulatory_set={LGPD|ISO|PCI|custom}`; `mask_pii=true|false`.

## Métricas e Critérios de Aceite
- Métricas: `latency_ms`, `cost.tokens_in/out`, `relevance`, `coverage`, `groundedness`, `diversity`.
- Aceite:
  - Cada item possui ao menos 1 citação válida (`doc_id/unit_id` ou `community_id`).
  - Reprodutibilidade com mesmos parâmetros/filtros.
  - Respeito a políticas de sensibilidade e RBAC.
  - Logs/traços por nó disponíveis (`/v1/queries/{id}/trace`).

## Rastreabilidade
- Roadmap: seções 8, 10, 11, 12 e 16 em `docs/roadmap.md`.
- Roadmap técnico: seções 1.3, 2.2, 3.1, 5, 6, 7 e 11 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Risco de custo elevado em projetos grandes → mitigar com `max_communities`, amostragem e cache.
- Hallucinations em respostas longas → mitigar com re-rank, citações obrigatórias e limites de contexto.
- Desalinhamento regulatório → parametrizar `regulatory_set` e validar mapeamentos.

## Exemplos
- API para geração:
  - `POST /v1/insights/generate`
  - Body (ex.):
```
{
  "project_id": "proj-123",
  "type": "risk",
  "filters": { "setor": "Financeiro", "lang": "pt" },
  "params": { "max_communities": 8, "k": 10, "rerank": true, "max_tokens_ctx": 6000 }
}
```
- Resposta (resumo):
```
{
  "id": "ins-789",
  "type": "risk",
  "payload": { "risks": [{ "title": "Lacuna de controle X", "severity": "alto", "evidence": [{"doc_id":"d1","unit_id":"u-42","page":3}] }] },
  "trace_id": "q-abc",
  "metrics": { "latency_ms": 1800, "tokens_in": 4200, "tokens_out": 600 }
}
```

## Variantes e Extensões
- Multimodal/cross-modal: incluir evidências de áudio (`time_range`, `speaker_label`) e imagem (`bbox`).
- Execução agendada: jobs periódicos por projeto/idioma com exportação automática.
- Integração social/organizacional: classificar riscos/tendências por equipes/processos vinculados.
