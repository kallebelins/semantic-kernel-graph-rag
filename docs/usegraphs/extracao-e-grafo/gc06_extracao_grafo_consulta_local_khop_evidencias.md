## UC06 — GraphCase: Consulta local k-hop com evidências citadas

Relacionado: `docs/usecases/uc06_extracao_grafo_consulta_local_khop_evidencias.md`

### Nós e papéis
- **PrepareSeedEntities**: Resolve entidades semente a partir da consulta.
- **ExpandKHop**: Expande subgrafo k-hop ao redor das sementes.
- **CollectEvidence**: Agrega evidências por aresta/nó com citações a `unit_id`.
- **RankSubgraph**: Reordena nós/arestas por relevância.
- **ReturnLocalAnswer**: Monta resposta local com subgrafo e citações.

### Arestas e condições
- `PrepareSeedEntities → ExpandKHop`: quando `seeds.count > 0`.
- `ExpandKHop → CollectEvidence`: quando `subgraph.nodes.length > 0`.
- `CollectEvidence → RankSubgraph`: quando `evidence.valid`.
- `RankSubgraph → ReturnLocalAnswer`: quando `ranking.valid`.

### Schemas de nós

#### PrepareSeedEntities
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "string" },
    "k": { "type": "number" }
  },
  "required": ["query", "k"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "seeds": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "entityIds": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["count", "entityIds"]
    }
  },
  "required": ["seeds"]
}
```

#### ExpandKHop
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "seeds": { "type": "object" },
    "k": { "type": "number" }
  },
  "required": ["seeds", "k"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "subgraph": {
      "type": "object",
      "properties": {
        "nodes": { "type": "array", "items": { "type": "string" } },
        "edges": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["nodes", "edges"]
    }
  },
  "required": ["subgraph"]
}
```

#### CollectEvidence
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "subgraph": { "type": "object" }
  },
  "required": ["subgraph"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "evidence": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "citations": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["valid"]
    }
  },
  "required": ["evidence"]
}
```

#### RankSubgraph
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "subgraph": { "type": "object" },
    "evidence": { "type": "object" }
  },
  "required": ["subgraph", "evidence"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "ranking": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "scores": { "type": "array", "items": { "type": "number" } }
      },
      "required": ["valid"]
    }
  },
  "required": ["ranking"]
}
```

#### ReturnLocalAnswer
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "subgraph": { "type": "object" },
    "ranking": { "type": "object" },
    "evidence": { "type": "object" }
  },
  "required": ["subgraph", "ranking"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "answer": {
      "type": "object",
      "properties": {
        "text": { "type": "string" },
        "citations": { "type": "array", "items": { "type": "object" } },
        "subgraphRef": { "type": "string" }
      },
      "required": ["text"]
    }
  },
  "required": ["answer"]
}
```


