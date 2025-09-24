## UC07 — GraphCase: Detecção hierárquica (Leiden/Louvain) por nível

Relacionado: `docs/usecases/uc07_comunidades_relatorios_deteccao_hierarquica.md`

### Nós e papéis
- **PrepareGraphView**: Define grafo de trabalho (projeção/ponderação/recorte).
- **DetectCommunities**: Executa detecção (Leiden/Louvain) com resolução por nível.
- **BuildHierarchy**: Consolida níveis e mapeia nós → comunidades.
- **PersistCommunityMeta**: Persiste metadados, níveis e parâmetros.

### Arestas e condições
- `PrepareGraphView → DetectCommunities`: quando `graphView.valid`.
- `DetectCommunities → BuildHierarchy`: quando `communities.levels > 0`.
- `BuildHierarchy → PersistCommunityMeta`: quando `hierarchy.valid`.

### Schemas de nós

#### PrepareGraphView
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "projection": { "type": "string" },
    "weightAttribute": { "type": "string" }
  },
  "required": ["projection"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "graphView": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" }
      },
      "required": ["valid"]
    }
  },
  "required": ["graphView"]
}
```

#### DetectCommunities
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "graphView": { "type": "object" },
    "method": { "type": "string", "enum": ["leiden", "louvain"] },
    "resolution": { "type": "number" }
  },
  "required": ["graphView", "method"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "communities": {
      "type": "object",
      "properties": {
        "levels": { "type": "number" },
        "assignments": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["levels", "assignments"]
    }
  },
  "required": ["communities"]
}
```

#### BuildHierarchy
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "communities": { "type": "object" }
  },
  "required": ["communities"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "hierarchy": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "levels": { "type": "number" }
      },
      "required": ["valid"]
    }
  },
  "required": ["hierarchy"]
}
```

#### PersistCommunityMeta
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "hierarchy": { "type": "object" },
    "parameters": { "type": "object" }
  },
  "required": ["hierarchy"]
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
        "persisted": { "type": "boolean" }
      },
      "required": ["persisted"]
    }
  },
  "required": ["persistResult"]
}
```


