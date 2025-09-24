## UC10 — GraphCase: Local GraphRAG (expansão por entidades)

Relacionado: `docs/usecases/uc10_consulta_local_graphrag_expansao_entidades.md`

### Nós e papéis
- **ResolveEntities**: Extrai/resolve entidades da consulta.
- **ExpandNeighborhood**: Expande vizinhança e coleta contexto local.
- **ComposeContext**: Constrói contexto textual a partir do subgrafo.
- **AnswerWithGraph**: Gera resposta citando evidências.

### Arestas e condições
- `ResolveEntities → ExpandNeighborhood`: quando `entities.length > 0`.
- `ExpandNeighborhood → ComposeContext`: quando `subgraph.nodes.length > 0`.
- `ComposeContext → AnswerWithGraph`: quando `context.valid`.

### Schemas de nós

#### ResolveEntities
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "string" }
  },
  "required": ["query"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "entities": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["entities"]
}
```

#### ExpandNeighborhood
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "entities": { "type": "array", "items": { "type": "string" } },
    "radius": { "type": "number" }
  },
  "required": ["entities"]
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

#### ComposeContext
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
    "context": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "text": { "type": "string" }
      },
      "required": ["valid", "text"]
    }
  },
  "required": ["context"]
}
```

#### AnswerWithGraph
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "context": { "type": "object" }
  },
  "required": ["context"]
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
        "citations": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["text"]
    }
  },
  "required": ["answer"]
}
```


