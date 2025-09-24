# Fusão/Merge no grafo (aliases, chaves de bloqueio, evidências)

## Contexto e Problema
Após a IE gerar `{entities[], relations[], claims[]}` por `TextUnit`, é necessário consolidar um grafo de conhecimento único por projeto. Sem merge controlado, o grafo sofre com duplicatas, arestas redundantes e perda de confiabilidade. É crítico preservar citações (`unit_id`) como evidências e operar de forma incremental e idempotente.

## Objetivo
Unificar entidades/relações/claims extraídos em nós/arestas canônicas do grafo do projeto, resolvendo aliases e aplicando chaves de bloqueio para reduzir comparações. Garantir:
- IDs estáveis por entidade/relação/claim em upserts repetidos (idempotência)
- Evidências preservadas e agregadas sem duplicação
- Incrementalidade (apenas difs aplicados) e integridade (constraints únicas)

## Atores e Escopo
- Usuários: Engenheiro de Conhecimento, Operador de Ingestão
- Serviços: `GraphMergeNode`, `IGraphStore` (Neo4j), Observabilidade
- Escopo: consolidação canônica e evidências. Não cobre detecção de comunidades nem consulta.

## Pré-condições
- Artefatos de IE disponíveis e válidos para um `projectId`
- Constraints únicas criadas no grafo (ver `docs/roadmap_tecnico.md` 3.2)
- Perfis de normalização e políticas de privacidade carregados

## Entradas
- Lote IE v1: `{entities[], relations[], claims[]}` com `evidence:[unit_id]`
- `projectId` e configuração de merge:
  - Dicionários de aliases opcionais por tipo
  - Estratégias de chave de bloqueio por tipo (ex.: `Company:{name_norm,country}`)
  - Limiares de confiança mínimos por estrutura

## Fluxo (alto nível)
- Normalizar texto (case folding, trim, locale) e nomes canônicos
- Gerar chaves de bloqueio por tipo para reduzir o espaço de comparação
- Resolver entidades candidatas via `{blockingKey → exact}` e, opcionalmente, heurísticas determinísticas (sem LLM)
- Upsert de entidades canônicas; agregar `aliases[]` e `mentions(unit_id)`
- Resolver relações: mapear `source`/`target` para `Entity.id` canônico e upsert por `(type, sourceId, targetId)` agregando `evidence`
- Resolver claims: `text_norm + polarity (+subject?)` → upsert agregando `evidence`
- Persistir arestas `MENTIONED_IN` e `SUPPORTED_BY` para manter rastreabilidade
- Emitir `graphChanges` (adds/updates) e `snapshotRef`
- Nós SKGraph: `GraphMergeNode` (+ `RetryPolicyGraphNode`, `ResultAggregator` em fan-out)

## Regras/Políticas e Guardrails
- Unicidade por projeto: `entity_id`, `relation_id`, `claim_id`
- Idempotência: reprocessar o mesmo lote não altera o grafo além da primeira aplicação
- Evidência obrigatória: relações/claims devem manter `evidence.unit_id` válido
- Sensibilidade/privacidade: respeitar `TextUnit.sensitivity` ao expor artefatos/traços
- Escopo de projeto: proibir merges cross-project

## Saídas e Artefatos
- `graphChanges`:
  - `entities {added[], updated[]}`
  - `relations {added[], updated[]}`
  - `claims {added[], updated[]}`
  - `edges {mentionedInAdded, supportedByAdded}`
- `snapshotRef` (versão do subgrafo após merge)
- Métricas/traços: comparações evitadas por bloqueio, dedup rate, latência, custo

## Parâmetros e Ajustes
- `min_confidence_entity|relation|claim`
- `blocking_keys_by_type` (ex.: `Person:{name_norm,org?}`; `Company:{name_norm,country}`)
- `batch_size`, `max_parallel`, `retry_policy`
- `evidence_merge_strategy`: `set_union` (default) | `append`

## Métricas e Critérios de Aceite
- Zero violações de constraints únicas sob carga típica
- Deduplicação: redução ≥ X% de duplicatas em amostras conhecidas
- Idempotência: aplicar o mesmo lote 2× produz `graphChanges = {}` na segunda execução
- Groundedness: 100% de `relations/claims` com `evidence.unit_id` válidos
- Performance: p95 latência por 10k estruturas dentro de orçamento definido

## Rastreabilidade
- Roadmap: 3) Modelo de Dados, 7) IE e Pós-Processo, 11) Operação/Incremental
- Roadmap técnico: 2.1 Ingestão (`Graph.Merge`), 3.2 Grafo (Neo4j), 18.3 I/O (`GraphMergeNode`)

## Riscos e Considerações
- Homônimos (pessoas/empresas) → reforçar chaves de bloqueio por contexto (org/país)
- Normalização agressiva pode colapsar entidades distintas → iniciar pelo exato; heurísticas conservadoras
- Evidências volumosas por nós populares → `set_union` e paginação de arestas
- Remoções/retrações: suportar marcação de obsolescência quando `Document` for removido

## Exemplos
Entrada (IE simplificada):
```json
{
  "entities": [
    {"type":"Organization","name":"Contoso S.A.","aliases":["Contoso"],"mentions":[{"unit_id":"u-123"}]},
    {"type":"Organization","name":"Fabrikam","mentions":[{"unit_id":"u-123"}]}
  ],
  "relations": [
    {"type":"Partnership","source":{"name":"Contoso S.A."},"target":{"name":"Fabrikam"},"evidence":["u-123"],"confidence":0.91}
  ],
  "claims": [
    {"text":"Parceria estabelecida em 2024","polarity":"positive","confidence":0.86,"evidence":["u-123"]}
  ]
}
```

Upserts esperados (conceitual):
```cypher
// Entidades (por projeto)
MERGE (e1:Entity {id:$contosoId, projectId:$projectId})
  ON CREATE SET e1.type='Organization', e1.name='Contoso S.A.', e1.aliases=['Contoso']
MERGE (e2:Entity {id:$fabrikamId, projectId:$projectId})
  ON CREATE SET e2.type='Organization', e2.name='Fabrikam'

// Relação canônica por (type, sourceId, targetId)
MERGE (e1)-[r:REL {id:$relId, type:'Partnership', projectId:$projectId}]->(e2)

// Evidências
MERGE (u:TextUnit {unitId:'u-123', projectId:$projectId})
MERGE (e1)-[:MENTIONED_IN]->(u)
MERGE (e2)-[:MENTIONED_IN]->(u)
MERGE (c:Claim {id:$claimId, projectId:$projectId})
  ON CREATE SET c.text='Parceria estabelecida em 2024', c.polarity='positive', c.confidence=0.86, c.evidence=['u-123']
MERGE (c)-[:SUPPORTED_BY]->(u)
```

Observação: `id` podem ser derivados de chaves estáveis (ex.: hash de `type+name_norm+country` para `Entity`, `type+sourceId+targetId` para `Relation`, `text_norm+polarity` para `Claim`) por projeto.

## Variantes e Extensões
- Integração social/organizacional: tipos `Person/Team/ProjectOrg` com chaves de bloqueio apropriadas
- Multimodal: agregar `evidence` com `time_range`/`bbox` além de `unit_id`
- Retratação/soft-delete: remover evidências e desativar nós/arestas quando fontes forem desindexadas
