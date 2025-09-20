# Padrão de saída Usecase: Cache Multicamadas

## Contexto e Problema
Consultas e artefatos recorrentes (respostas, relatórios de comunidade, features de re-rank, subgrafos) geram custos repetidos. É necessário um cache multicamadas, seguro e ciente de políticas, com chaves determinísticas e invalidação seletiva para reduzir latência e custo.

## Objetivo
Diminuir custo e tempo de respostas utilizando caches para: respostas finais, `CommunityReport`, embeddings/resultados de QFS, vizinhanças locais (k-hop), e features do `ReRanker`. Suportar pré-aquecimento por projeto e expiração/invalidação sob mudanças.

## Atores e Escopo
- Usuários: Todos os consumidores de consulta e insights.
- Serviços: `CacheService` (L1 in-memory, L2 Redis/Elasticache, L3 object storage), `GlobalQFS`, `LocalGraph`, `ReRanker`.
- Fora do escopo: CDN/edge cache externo (pode ser extensão futura para UI).

## Pré-condições
- Definição de políticas por tipo de artefato (`ttl`, `max_size`, `sensitivity_threshold`).
- Namespaces por `tenant/project` e versionamento de DTOs (influencia chaves).

## Entradas
- Chaves de cache por tipo:
  - Resposta: `{projectId, questionHash, mode, filtersHash, paramsHash, version}`
  - QFS: `{projectId, queryHash, level, maxCommunities, version}`
  - Local (k-hop): `{projectId, seedEntitiesHash, depth, beam, filtersHash, version}`
  - CommunityReport: `{projectId, communityId, level, reportVersion}`
  - ReRank features: `{projectId, candidatesHash, modelVersion}`
- Parâmetros: `ttl`, `stale_while_revalidate`, `max_entries`, `policy={LRU|LFU|ARC}`.

## Fluxo (alto nível)
1) Ao iniciar um nó, `CacheService` verifica L1→L2→L3 por chave.
2) Hit: retorna valor e registra métricas; opcional SWR para renovar em background.
3) Miss: executa nó, persiste em L3→L2→L1 conforme política e retorna resultado.
4) Invalidação seletiva por eventos (`GraphUpdated`, `ReportsGenerated`, `ContentIngested`) e por escopos (project/source/community).

- Nós SKGraph (exemplos): wrappers de nós com `RetryPolicyGraphNode` + `FunctionGraphNode(CacheGet/Set)`; `ResultAggregator` preservando metadados de origem.

## Regras/Políticas e Guardrails
- Respeitar RBAC/ABAC e sensibilidade: nunca servir artefatos acima do limiar do usuário; chaves incluem `policyVersion`.
- Idempotência e determinismo: serialização estável para hashes.
- Evitar cache poisoning: assinar entradas com `project/tenant` e versões de modelo.

## Saídas e Artefatos
- Entradas de cache por camada (in-memory, Redis, object storage) com manifest e métricas.
- Tabelas/índices de uso para BI de operação.

## Parâmetros e Ajustes
- `ttl` por tipo, `max_entries` por camada, `policy`, `swr_ms`.
- `prewarm={community_reports, intents_examples, top_queries}`.

## Métricas e Critérios de Aceite
- Métricas: hit ratio por camada, latência média e p95, economia de tokens, taxa de invalidação.
- Aceite: hit ratio melhora após aquecimento; nenhuma violação de políticas; queda de custo comprovada em A/B.

## Rastreabilidade
- Roadmap: seção 11 em `docs/roadmap.md` (Cache Multicamadas).
- Roadmap técnico: seções 7 e 9 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Stale data após mudanças: usar versões e eventos de invalidação; SWR curto.
- Chaves grandes/lentas: otimizar hashing e normalizar parâmetros.
- Multi-tenant: isolar namespaces e quotas.

## Exemplos
- Política de cache (conceito):
```
{
  "project_id": "proj-123",
  "policy": {
    "answer": { "ttl_ms": 900000, "max_entries": 5000, "layer": ["L1","L2"] },
    "community_report": { "ttl_ms": 259200000, "layer": ["L2","L3"] }
  }
}
```

## Variantes e Extensões
- Cache distribuído com consistência eventual e CRDTs.
- Cache de prompts/few-shots e de embeddings de intent.
