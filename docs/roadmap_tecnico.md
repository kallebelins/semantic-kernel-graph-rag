# Roadmap Técnico

Este documento consolida, em formato de referência rápida, todos os elementos técnicos citados no roadmap e documentos auxiliares: interfaces/classes (.NET + SKGraph), nós de pipeline (ingestão/consulta), modelos de dados (SQL/Neo4j), endpoints de API, parâmetros, serviços e metadados de citações multimodais.

---

## 0) Princípios e Diretrizes

- **IE estruturada**: extração de entidades, relações e afirmações com esquema padronizado por domínio e citação explícita de unidades de evidência.
- **Grafo de conhecimento**: consolidação com resolução de aliases, fusão por chaves de bloqueio e manutenção de evidências; resumos por entidade em múltiplos níveis.
- **Comunidades hierárquicas**: detecção multi-nível (fino → macro) para navegação e sumarização bottom-up.
- **Modos de consulta complementares**: local (k-hop), global (QFS) e híbrido (DRIFT), com roteamento por intenção.
- **Seleção dinâmica orientada a custo**: limites de tokens, profundidade e abertura incremental de contexto por ganho esperado.
- **Observabilidade e governança**: traços por etapa, métricas de qualidade e custos, com políticas de segurança, privacidade e auditoria por projeto/tenant.

## 1) Interfaces e Classes (.NET + SKGraph)

### 1.1 Interfaces principais (ingestão, grafo, comunidades, relatório)

```csharp
public interface IIEExtractor {
  Task<ExtractionResult> ExtractAsync(TextUnit unit, PromptProfile profile);
}

public interface IGraphStore {
  Task UpsertEntitiesAsync(IEnumerable<Entity> e);
  Task UpsertRelationsAsync(IEnumerable<Relation> r);
  Task UpsertClaimsAsync(IEnumerable<Claim> c);
  Task<GraphSnapshot> GetSubgraphAsync(string projectId);
}

public interface ICommunityService {
  Task<List<Community>> DetectAsync(GraphSnapshot g, CommunityAlgo algo, int level);
}

public interface IReportService {
  Task<CommunityReport> SummarizeAsync(Community c, int level, PromptProfile profile);
}
```

### 1.2 Orquestração de grafos (SKGraph)

- Composição por nós `IGraphNode` e execução via `GraphExecutionContext`/`GraphState`.
- Nós utilitários do SKGraph:
  - `SwitchGraphNode` (roteamento por intenção/heurística)
  - `ConditionalGraphNode` (condições sobre o estado)
  - `RetryPolicyGraphNode` (aplica política de retry idempotente)
  - `SubgraphGraphNode` (aninhamento de subgrafos)
  - `FunctionGraphNode` (vincula funções do Kernel)
  - `ForeachLoopGraphNode`/`WhileLoopGraphNode` (fan-out/loops controlados)
- Arestas/fluxo:
  - `ConditionalEdge` e `TemplateConditionalEdge` para transições condicionais
  - Agregação via `ResultAggregator` quando houver fan-out
  - Pausa/retomada e governança via `GraphExecutionOptions`

### 1.3 Serviços/Plugins SK suportados

- Ingestão e indexação: `Chunker`, `Embedder`, `BM25` (ou `Bm25Indexer`)
- Extração e grafo: `IEExtractor`, `GraphStore/GraphMerge`
- Comunidades e relatório: `CommunityDetection`, `ReportSummarizer`
- Resposta/Qualidade: `ReRanker`, `Guardrails`, `Insights`
- Implementação (.NET + SK): `EmbeddingService`, `BM25Service`, `GraphService`, `Summarizer`, `PIIFilter`, `InsightPlaybooks`

### 1.4 Arquitetura em Camadas

- **Gateway de API**: REST/JSON (opcional gRPC), autenticação e autorização; recebe uploads, inicia sincronizações, expõe consultas e insights.
- **Núcleo de Serviço**: orquestração de pipelines como DAGs (SKGraph) com nós especializados e roteamento por intenção.
- **Workers de Ingestão**: execução assíncrona com checkpointing e reprocessamento incremental por fonte.
- **Camada de Conhecimento**:
  - Armazenamento de objetos para originais e artefatos.
  - Armazenamento vetorial (trechos/relatórios) e busca lexical (BM25).
  - Banco de grafo (entidades, relações, comunidades, relatórios) e banco relacional (metadados, auditoria, métricas).
- **Observabilidade**: logs estruturados, métricas e traços por nó, correlacionados por consulta/projeto/usuário.
- **Console Administrativo**: gestão de tenants, taxonomias, prompts, políticas e recursos.

---

## 2) Nós de Pipeline (DAG)

### 2.1 Ingestão

- Detect → Normalize → Chunk → Embed (+BM25)
- IE.Extract → Graph.Merge
- CommunityDetect(level=0..n) → Report.Summarize(level=0..n)
- AutoFewShots (opcional)
- Guardrails/Policies → Done

#### Estratégias de Fragmentação e Indexação

- **Heading-aware**: respeita hierarquias de títulos (níveis 1–4) para preservar coerência temática.
- **Semantic split**: delimitação por coesão semântica (sinais linguísticos e embeddings de sentenças).
- **Table/Code aware**: preservação de tabelas e trechos técnicos com contexto adjacente.
- **Rolling window híbrida**: janelas sobrepostas para continuidade e redução de cortes duros.
- **Configuração por projeto**: tamanho máximo, overlap, tamanho mínimo de sentença e preservação de listas/tabelas.
- **Indexação vetorial**: coleções por tenant/projeto com payloads de metadados (documento, unidade, taxonomia, idioma).
- **BM25**: índice lexical com campos facetados para filtros e recuperação rápida.

#### IE Pós-processo

- **Esquemas normalizados** por domínio e formatos de saída consistentes com citação obrigatória (`unit_id`).
- **Alias e normalização**: case folding, lematização e regras específicas por tipo para unificar referências.
- **Chaves de bloqueio**: combinações de atributos para reduzir comparações ao mesclar entidades/relações.
- **Passada de gleanings**: segunda rodada seletiva em unidades densas para aumentar recall com custo controlado.

### 2.2 Consulta (Roteamento por Intent)

- `DetectIntent` → arestas para `{ VectorRAG, LocalGraphRAG, GlobalGraphRAG, DRIFT, InsightOnly }`
  - Guards: confiança do classificador, presença de termos (“panorama”, “tendência”, “comparativo”), tamanho da pergunta, etc.
- Heurísticas:
  - Perguntas “panorama/visão geral/tendências/mapa” → GlobalQFS
  - Fato específico curto → VectorRAG (ou LocalGraph quando envolve pessoas/sistemas/regulação)
  - Ambígua/longa → DRIFT

### 2.3 Comunidades (Leiden/Louvain)

- Serviço opcional (Python, `leidenalg/igraph`):

```csharp
var parts = await _http.PostJsonAsync<List<Partition>>(
  "http://leiden-svc/communities",
  new { edges = snapshot.Edges, resolution = 1.0, iterations = 10 }
);
```

- Fallback .NET: Louvain + refinamento (conductance/triangles)

#### Métricas e Representações

- **Métricas de particionamento**: modularidade e qualidade de corte; refinamentos locais opcionais.
- **Representações para QFS**: embeddings de `CommunityReport` para busca global focada em consulta (map-reduce).

---

## 3) Modelos de Dados

### 3.1 Relacional (SQL)

- `Project`: `project_id`, `tenant_id`, `name`, `taxonomy {tags[], setores[], temas[]}`, `policies`, `rag_modes_enabled[]`
- `Source`: `source_id`, `type {upload,gdrive,github,zip,http}`, `uri`, `status`, `last_sync`
- `Document`: `doc_id`, `project_id`, `source_id`, `filename`, `mime`, `hash`, `tags[]`, `setor`, `tema`, `lang`, `status`
- `TextUnit`: `unit_id`, `doc_id`, `order`, `text`, `tokens`, `metadata {heading,page,section,tag/setor/tema,lang}`, `embedding_id`
- `EmbeddingIndex`: `embedding {id, vector, ref=unit/community_report/intent_examples}`
- `BM25Index`: `bm25 {unit_id -> postings}`
- `NLP Config`: `prompt_profiles {extraction,summarization,answering,classification}`, `few_shots {task, examples[]}`, `intent_schemas {name, patterns, classifiers, router_graph}`
- `Query`: `id`, `project_id`, `user_id`, `text`, `mode`, `params`, `cost`, `latency`
- `Insight`: `id`, `type`, `payload`, `score`, `linked_query_ids[]`

### 3.2 Grafo (Neo4j)

Constraints (Cypher):

```cypher
CREATE CONSTRAINT entity_id   IF NOT EXISTS FOR (e:Entity)    REQUIRE e.id IS UNIQUE;
CREATE CONSTRAINT relation_id IF NOT EXISTS FOR (r:Relation)  REQUIRE r.id IS UNIQUE;
CREATE CONSTRAINT claim_id    IF NOT EXISTS FOR (c:Claim)     REQUIRE c.id IS UNIQUE;
CREATE CONSTRAINT community_id IF NOT EXISTS FOR (m:Community) REQUIRE m.id IS UNIQUE;
```

Nós/Propriedades:
- `(:Entity {id, type, name, aliases:[], projectId, summaries:{lvl0,lvl1,...}})`
- `(:Claim  {id, text, polarity, confidence, evidence:[unitId], projectId})`
- `(:Relation {id, type, projectId})`
- `(:Community {id, level, projectId, centroidEntities:[], summary})`
- `(:TextUnit {unitId, docId, tags, setor, tema, lang})`

Arestas/Propriedades:
- `(:Entity)-[:MENTIONED_IN]->(:TextUnit)`
- `(:Entity)-[:REL]->(:Entity)`
- `(:Claim)-[:SUPPORTED_BY]->(:TextUnit)`
- `(:Entity)-[:IN_COMMUNITY {level}]->(:Community)`

#### 3.2.1 Camada Social/Organizacional (opcional)

Novos tipos e relações para integrar pessoas/processos:
- Nós: `(:Person {id, name, role, dept})`, `(:Team {id, name})`, `(:ProjectOrg {id, name})`
- Arestas: `(:Person)-[:PARTICIPATED_IN]->(:ProjectOrg)`, `(:Person)-[:MEMBER_OF]->(:Team)`, `(:Person)-[:MENTIONED_IN]->(:TextUnit)`, `(:Person)-[:REL {type}]->(:Entity)`

---

## 4) Citações e Metadados Multimodais

- Chaves e metadados de evidência: `unit_id`, `doc_id`, `page`, `section`, `heading`, `lang`
- Mídia:
  - Áudio/Vídeo: `time_range` (ex.: `00:12:40-00:13:05`), `speaker_label`
  - Imagem: `bbox` (região OCR)
  - Documento: `page`, `section`, `heading`
- Governança: `sensitivity` (Público | Interno | Confidencial), `tags`, `setor`, `tema`

---

## 5) Modos de Consulta e Parâmetros

- `Vector RAG`: `(k1 BM25 ∪ k2 ANN)` → `ReRanker` → síntese; filtros por `tags/setor/tema/lang`
- `Local GraphRAG`: expansão k-hop a partir de entidades relevantes → coleta de *TextUnit* de evidência → síntese com citações
- `Global GraphRAG (QFS)`: seleção e navegação por `Community[level]` + map-reduce de `CommunityReport`
- `DRIFT (Híbrido)`: filtro global → subperguntas locais → re-rank final
- Parâmetros de custo/seleção: `k`, `rerank`, `max_tokens_ctx`, `max_communities`, `beam_width`, `depth`

Detalhes adicionais:
- **Consultas cross-modais**: incluir `Intent.CrossModal` no roteador; prompts adaptados para fundir citações heterogêneas (áudio/imagem/texto); reclassificação/ordenador unificado; filtros por taxonomia e sensibilidade por mídia.

---

## 6) Endpoints de API

Autenticação (Keycloak/OAuth2): scopes `read:rags`, `write:ingest`, `admin:projects`

- Ingestão:
  - `POST /v1/projects`
  - `POST /v1/projects/{id}/sources`
  - `POST /v1/projects/{id}/upload` (multipart/zip)
  - `POST /v1/projects/{id}/sync`
  - `GET  /v1/jobs/{jobId}`
- Configuração:
  - `PUT /v1/projects/{id}/taxonomy`
  - `PUT /v1/projects/{id}/rag-config`
  - `PUT /v1/projects/{id}/prompts`
  - `PUT /v1/projects/{id}/intents`
- Consulta:
  - `POST /v1/query` – body inclui `project_id`, `question`, `mode`, `filters {tags,setor,tema,lang}`, `params {k,rerank,max_tokens_ctx,max_communities}`
- Insights:
  - `POST /v1/insights/generate`
  - `GET  /v1/insights?project_id=...&type=risk|trend|faq|gap`
- Observabilidade:
  - `GET /v1/queries/{id}/trace`
  - `GET /v1/metrics`

---

## 7) Observabilidade, Qualidade e A/B

- Traços por nó (SKGraph): `latency_ms`, `cost {input_tokens, output_tokens}`, `provider/model`
- Métricas de qualidade: `relevance`, `coverage`, `groundedness`, `diversity`, `empowerment`
- Benchmarks: classes de pergunta `{local fact, procedimento, comparativo, panorama, compliance}`
- A/B por modo/intent: Vector vs Local vs Global vs DRIFT
- Drift monitor: mudança de vocabulário por `setor/tema`

Adições:
- **Trace XAI**: serialização do subgrafo/trace por consulta em JSON (nós, arestas, passos, métricas) e renderização opcional (Mermaid/Neo4j Bloom/GraphView) via console.
- **Benchmarks internos**: conjuntos estratificados por classes de pergunta com verificação automatizada.

---

## 8) Segurança, LGPD e Governança

- Classificação de sensibilidade por `TextUnit`
- Redação/Mascaramento de PII conforme escopo e perfil do usuário
- RBAC/ABAC por `tenant/project/tag/setor/tema`
- Direito ao esquecimento por `docId/unitId/entityId`
- Citações obrigatórias (com `docId/unitId` ou `communityId`) para respostas decisórias

---

## 9) Operação e Custos

- Orçamento por consulta: `max_tokens_ctx`, `max_steps`, `max_level`
- Seleção dinâmica de comunidades/contexto (abre níveis sob demanda)
- Cache multicamadas: respostas, `CommunityReport`, features de re-rank
- Indexação incremental (diff por fonte) e reprocessamento seletivo
- Multi-tenant: separação por `tenant/project` em índices, grafos e storage

---

## 10) Serviços, Stores e Dependências

- Graph DB: Neo4j (labels `Entity{type}`, `Claim`, `Relation`, `Community{level}`)
- Vector Store: Qdrant/pgvector/Weaviate (HNSW; cosine/dot)
- BM25/Lexical: OpenSearch/Elasticsearch/Lucene
- Object Storage: S3/Azure Blob/GCS (`raw/`, `normalized/`, `artifacts/`, `audits/`)
- SQL: Postgres/SQL Server (metadados, auditoria, quotas, feature flags)
- Auth: Keycloak/OAuth2
- Comunidades (opcional): microserviço Python `leidenalg/igraph` (`POST http://leiden-svc/communities`)
- Re-Ranker: cross-encoder (MLM) para ordenação final

---

## 11) Playbooks de Insights (implementáveis)

- Riscos & Lacunas: varredura de comunidades ligadas a LGPD/ISO
- Panorama Executivo: top-5 temas por setor, tendências mês a mês
- FAQ Generator: perguntas frequentes por tema com respostas citadas
- Matriz de Conformidade: mapeia `claims` → requisitos (LGPD/ISO/PCI)

- **BI Integrado**: exportação de entidades/claims e indicadores para Data Lake/Elastic; dashboards (PowerBI/Tableau); execução de playbooks temáticos por projeto.

---

## 12) Prompts e Few-Shots

- Perfis de prompt: `extraction`, `summarization`, `answering`, `classification`
- Auto Few-Shot Tuning: amostragem estratificada por `setor/tema/tag`; geração/validação (balanceamento, dedup, toxicity/PII); versionamento por projeto/idioma
- Exemplos (objetivos/resumo):
  - Extraction (IE): JSON com `{entities[], relations[], claims[]}` citando `unit_id`
  - Community Report: objetivos, riscos, processos, dependências, “3 coisas a saber”, “3 lacunas”, citando `community_id`
  - QFS (Global): “mapa mental” via community reports mais relevantes
  - Answering (Local): resposta curta (≤8 linhas) com citações `doc_id/unit_id`

Evolução dinâmica:
- **Treinamento dinâmico de few-shots (auto evolução)**: reaproveita Q/A validados e respectivos traces para gerar/curar exemplos; balanceamento, deduplicação e higiene; versionamento por projeto/idioma.

---

## 13) Parâmetros/Filtros suportados (consulta)

- `filters`: `{tags[], setor, tema, lang}`
- `params`: `{k, rerank, max_tokens_ctx, max_communities, beam_width, depth}`
- `mode`: `auto | vector | local_graph | global_graph/qfs | drift`
- `intent` (exemplos): `{FAQ, Procedimento, Compliance, RH, Jurídico, Vendas, SuporteN1, Estratégia, Panorama/Trends, CrossModal}`

---

## 14) Diferenciais multimodais (campos-chave)

- Áudio/Vídeo: `time_range`, `speaker_label`
- Imagem: `bbox`
- Documento: `page`, `section`, `heading`
- Normalização e persistência desses metadados desde a ingestão → resposta → UI

---

## 15) Roadmap (ancoras técnicas)

- Fase 1 (MVP): Upload/ZIP/GDrive → Normalize/Chunk → Embeddings + BM25 → Vector RAG (API) + Taxonomia/Auth/Logs
- Fase 2 (Grafo & Local): IE + GraphMerge (Neo4j) → LocalGraph (k-hop) com citações
- Fase 3 (Global & DRIFT): Comunidades (Leiden/Louvain) + Community Reports (níveis) → GlobalQFS + DRIFT + seleção dinâmica
- Fase 4 (Auto-tuning & Insights): Few-shots automáticos + Playbooks + A/B & custos

---

## 16) Diferenciais Disruptivos

- **Citações multimodais contextuais**
  - Implementável: propagar `unit_id`, `time_range`, `bbox`, `page`, `section` do pipeline à resposta/UI; normalizar formatos por mídia; persistir nas unidades e nas estruturas de citação.

- **Consultas cross-modais**
  - Implementável: `Intent.CrossModal` no roteador; prompts para fundir citações de mídias distintas; reclassificação/ordenação unificada; filtros por taxonomia/sensibilidade por mídia.

- **RAG explicável (XAI + grafo)**
  - Implementável: serializar subgrafo/trace em JSON; expor via API; renderização em Mermaid/Neo4j Bloom/GraphView.

- **Agente proativo com detecção de eventos**
  - Implementável: watchers e classificadores por janelas de tempo; queries contínuas no grafo; publicação (Teams/Slack/Webhook) com citações.

- **Treinamento dinâmico de few-shots (auto evolução)**
  - Implementável: persistir Q/A + trace; curadoria (balanceamento, dedup, higiene); versionar e aplicar `PromptProfiles` por projeto.

- **Integração com grafos de pessoas/processos**
  - Implementável: novos tipos de entidade/relação (participou de, aprovou, discutiu); mapeamento para departamentos/processos; consultas por envolvimento/impacto.

- **Governança adaptativa**
  - Implementável: aplicar políticas ao montar prompts/exibir; redigir/mascarar por sensibilidade; auditoria por usuário/consulta/resultado.

- **RAG + métricas de negócio (BI integrado)**
  - Implementável: jobs de exportação para Data Lake/Elastic; dashboards; playbooks (tendências, riscos, FAQs) por projeto.

---

## 17) Compatibilidade com SemanticKernel.Graph (Requisitos)

- **Builder/DAG**: uso de `GraphBuilder` para definição de nós/arestas nomeados; roteamento compatível com `IntentClassifier` conforme exemplos da biblioteca.
- **Contexto**: `GraphContext` propagado entre nós contendo `projectId`, `traceId`, `cancellationToken`, `budget` e `artifacts` (imutáveis ou versionados por etapa).
- **Contratos de nó**: métodos `Run(ctx, ...)` assíncronos, determinísticos e idempotentes; aceitam `CancellationToken`; expõem métricas de custo/latência via hooks de telemetria do SK.
- **Tratamento de falhas**: política de retry exponencial idempotente; timeouts por nó; circuit-breaker por fornecedor/serviço; erros serializados para observabilidade.
- **Versionamento**: `NodeInput/NodeOutput` versionados (ex.: `v1`, `v2`) com compatibilidade retroativa; migrações controladas por projeto.
- **Segurança**: passagem de escopo e políticas (RBAC/ABAC) no contexto; filtros aplicados antes de montar prompts e outputs.
- **DI/Config**: injeção via contêiner padrão (.NET) com factories para provedores (LLM, vetorial, BM25, Neo4j) e chaves por tenant/projeto.

---

## 18) Contratos de Nós e Arestas (DAG/SKGraph)

### 18.1 Contratos de Nó

- Assinatura sugerida (.NET):

```csharp
public interface IGraphNode<TInput, TOutput>
{
  Task<TOutput> RunAsync(GraphContext context, TInput input, CancellationToken cancellationToken);
}
```

- Requisitos:
  - Idempotência por chave `{projectId, nodeName, inputHash}` com cache opcional.
  - Determinismo para mesma entrada/configuração.
  - Telemetria: `latency_ms`, `cost.tokens_in/out`, `provider/model`.
  - Orçamento: respeitar `context.Budget` (`max_tokens_ctx`, `max_steps`, `max_level`).
  - Saída: `TOutput` serializável, com `evidenceRefs` quando aplicável.

### 18.2 Tipos de Aresta e Semântica

- `Edge(A,B)`: sequência simples (default).
- `Edge(A,B, when: predicate)`: condicional por intenção/parâmetros/heurística.
- `FanOut(A, {B,C}, strategy: parallel)`: paralelismo controlado; `Join({B,C}, D, reducer)` para agregação.
- `Retry(A, policy)`: semântica de reexecução idempotente.
- Propagação de falhas: `FailFast` (aborta) ou `Compensate` (nó de compensação declarado).

### 18.3 Contratos I/O por Nó-base

- `Normalizer.Run`: `Input{sourceId, rawBytes|uri}` → `Output{documents[], textUnits[]}`
- `Chunker.Run`: `Input{textUnits[], strategy}` → `Output{textUnits[]}`
- `Embedder.Run`: `Input{textUnits[]}` → `Output{embeddings[], indexRefs}`
- `Bm25Indexer.Run`: `Input{textUnits[]}` → `Output{postingsRef}`
- `IEExtractorNode.Run`: `Input{textUnits[]}` → `Output{entities[], relations[], claims[]}`
- `GraphMergeNode.Run`: `Input{entities[], relations[], claims[]}` → `Output{graphChanges, snapshotRef}`
- `CommunityDetect.Run(level)`: `Input{snapshotRef|edges}` → `Output{partitions, metrics}`
- `ReportSummarize.Run(level)`: `Input{communityIds}` → `Output{reports[], embeddings[]}`
- `GlobalQFS/LocalGraph/VectorRag`: `Input{question, filters, params}` → `Output{answer, citations[], trace}`

---

## 19) Boas Práticas de Arquitetura (DDD)

- **Bounded Contexts**: `Ingestao`, `Indexacao`, `IE`, `Grafo`, `Comunidades`, `Relatorio`, `Consulta`, `Insights`, `Governanca`.
- **Aggregates**: `Project`, `Source`, `Document`, `TextUnit`, `Entity`, `Relation`, `Claim`, `Community`, `CommunityReport`, `Query`, `Insight`.
- **Repos/Ports**: `IGraphStore`, `IVectorIndex`, `ILexicalIndex`, `IObjectStorage`, `IProjectRepository` (ports); adapters por fornecedor.
- **Eventos de Domínio**: `ContentIngested`, `GraphUpdated`, `CommunitiesDetected`, `ReportsGenerated`, `QueryAnswered` para integração eventual e cache.
- **UoW e Consistência**: transações por agregado; outbox para integrações externas; consistência eventual entre índices e grafo.
- **ACL/Anti-Corrupção**: camadas de tradução entre modelos externos (LLM/providers) e o domínio.
- **Especificações/Consultas**: encapsular filtros (`tags/setor/tema/lang`) e políticas em objetos de especificação.
- **Invariantes**: citações obrigatórias para respostas decisórias; constraints de unicidade (`entity_id`, `claim_id`, etc.).
- **Resiliência**: retries idempotentes, timeouts, circuit-breakers, bulkheads por contexto; política de backoff por fornecedor.
- **Observabilidade**: correlação por `traceId`/`queryId`/`projectId`; logs estruturados; amostragem configurável.
- **Versionamento de Schemas**: DTOs versionados e migrações compatíveis por projeto/idioma.

---

## 20) Referências de Código e Exemplos

- Biblioteca: `https://www.nuget.org/packages/SemanticKernel.Graph`
- Exemplos: `https://github.com/kallebelins/semantic-kernel-graph-docs/tree/main/examples`
- Documentação: `https://skgraph.dev/`
