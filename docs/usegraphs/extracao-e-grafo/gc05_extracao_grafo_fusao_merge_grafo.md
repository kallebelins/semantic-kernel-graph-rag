## UC05 — GraphCase: Fusão/Merge no grafo (aliases, chaves de bloqueio, evidências)

Relacionado: `docs/usecases/uc05_extracao_grafo_fusao_merge_grafo.md`

### Nós e papéis
- **LoadCandidates**: Carrega candidatos a merge por alias/chave de bloqueio.
- **ResolveAliases**: Calcula similaridade e decide merges propostos.
- **MergeNodesEdges**: Executa merge de nós/arestas preservando evidências e histórico.
- **PersistMerged**: Aplica operações no grafo e registra auditoria.

### Arestas e condições
- `LoadCandidates → ResolveAliases`: quando `candidates.count > 0`.
- `ResolveAliases → MergeNodesEdges`: quando `proposals.assigned`.
- `MergeNodesEdges → PersistMerged`: quando `mergePlan.valid`.

### Schemas de nós

#### LoadCandidates
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "projectId": { "type": "string" },
    "blockingKeys": { "type": "array", "items": { "type": "string" } }
  },
  "required": ["projectId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "candidates": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "pairs": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["count", "pairs"]
    }
  },
  "required": ["candidates"]
}
```

#### ResolveAliases
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "candidates": { "type": "object" },
    "threshold": { "type": "number" }
  },
  "required": ["candidates"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "proposals": {
      "type": "object",
      "properties": {
        "assigned": { "type": "boolean" },
        "items": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["assigned"]
    }
  },
  "required": ["proposals"]
}
```

#### MergeNodesEdges
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "proposals": { "type": "object" },
    "preserveEvidence": { "type": "boolean" }
  },
  "required": ["proposals"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "mergePlan": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "operations": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["valid"]
    }
  },
  "required": ["mergePlan"]
}
```

#### PersistMerged
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "mergePlan": { "type": "object" }
  },
  "required": ["mergePlan"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "persistResult": {
      "type": "object",
      "properties": {
        "persisted": { "type": "boolean" },
        "applied": { "type": "number" }
      },
      "required": ["persisted"]
    }
  },
  "required": ["persistResult"]
}
```


