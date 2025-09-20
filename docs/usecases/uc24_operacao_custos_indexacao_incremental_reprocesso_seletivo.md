# Padrão de saída Usecase: Indexação Incremental e Reprocesso Seletivo

## Contexto e Problema
Novos conteúdos e alterações parciais exigem reprocessamento eficiente. Reexecutar todo o pipeline é caro e lento. É necessário detectar diffs por fonte e reprocessar seletivamente apenas unidades afetadas, preservando consistência entre índices, grafo e relatórios.

## Objetivo
Habilitar sincronizações incrementais e reprocessos seletivos orientados a diffs (por `source`, `document`, `unit`), garantindo consistência eventual entre vetorial, BM25, grafo, comunidades e relatórios, com observabilidade e governança.

## Atores e Escopo
- Usuários: Operadores de dados, donos de projeto.
- Serviços: `IngestionWorker`, `Normalizer`, `Chunker`, `Embedder`, `Bm25Indexer`, `IEExtractorNode`, `GraphMergeNode`, `CommunityDetect`, `ReportSummarize`.
- Fora do escopo: Integrações específicas de conectores além de upload/zip/gdrive/http.

## Pré-condições
- Projeto e fontes cadastrados; última sincronização registrada.
- Estratégia de detecção de diffs por fonte (hash/timestamp/etags/listing) definida.
- Políticas de direito ao esquecimento ativas (apagas seletivas devem propagar).

## Entradas
- `project_id`, `source_id?`, `doc_ids?`, `units?`, `since?`, `strategy={hash|mtime|manifest}`.
- Parâmetros: `parallelism`, `batch_size`, `gleanings=true|false`, `max_retries`, `levels_to_update={entities|relations|claims|communities|reports}`.

## Fluxo (alto nível)
1) Detectar diffs: listar/baixar manifest; comparar hashes/mtimes e identificar `added|modified|deleted`.
2) Normalizar e fragmentar apenas documentos alterados; calcular deltas de `TextUnit`.
3) Reindexar: embeddings e BM25 para unidades novas/alteradas; remover postings e vetores obsoletos.
4) IE seletiva nas unidades impactadas; `GraphMerge` com chaves de bloqueio e manutenção de evidências (adicionar/remover).
5) Comunidades/Relatórios: recalcular apenas partições afetadas (níveis) e atualizar `CommunityReport`/embeddings.
6) Emitir eventos (`ContentIngested`, `GraphUpdated`, `ReportsGenerated`) e atualizar `last_sync`.

- Nós SKGraph (exemplos): `ForeachLoopGraphNode(Diffs)`, `RetryPolicyGraphNode`, `SubgraphGraphNode(IE+Graph)`, `FunctionGraphNode(DeltaIndex)`, `ResultAggregator`.

## Regras/Políticas e Guardrails
- Respeitar quotas e janelas de sincronização por projeto/tenant.
- RBAC/ABAC: escopo de fontes/documentos autorizado.
- Idempotência por chave `{projectId, sourceId, docHash}`; outbox para integrações externas.

## Saídas e Artefatos
- `Job` de sincronização com contagens `{added, modified, deleted}` e artefatos em `artifacts/normalized/`, `artifacts/exports/`.
- Índices atualizados (vetorial/BM25), grafo consistente, relatórios reprocessados conforme impacto.

## Parâmetros e Ajustes
- `since` e `strategy` para incremental; `gleanings` para rodadas seletivas em unidades densas.
- `levels_to_update` para limitar recomputação de comunidades/relatórios.
- `parallelism`, `batch_size`, `max_retries`.

## Métricas e Critérios de Aceite
- Métricas: tempo de sync, custo, throughput, taxa de alteração, impacto no grafo (nós/arestas tocados).
- Aceite: consistência observável; índices sem órfãos; relatórios atualizados onde necessário; traços por etapa disponíveis.

## Rastreabilidade
- Roadmap: seção 11 (Operação, custos e escala) em `docs/roadmap.md`.
- Roadmap técnico: seções 2.1, 2.2, 3, 7, 9 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Explosão de recalculações de comunidade: limitar por `levels_to_update` e heurísticas de impacto.
- Remoções exigem limpeza cuidadosa de evidências e relações.
- Consistência eventual entre camadas → usar eventos e reconciliação periódica.

## Exemplos
- Solicitar sync incremental:
```
POST /v1/projects/{id}/sync
{
  "source_id": "src-42",
  "since": "2025-09-01T00:00:00Z",
  "params": { "strategy": "hash", "parallelism": 8, "gleanings": true, "levels_to_update": ["entities","claims"] }
}
```

## Variantes e Extensões
- Reprocesso seletivo por `entityId/claimId` para auditorias/correções.
- Previews/dry-run com estimativas de custo/tempo antes de executar.
