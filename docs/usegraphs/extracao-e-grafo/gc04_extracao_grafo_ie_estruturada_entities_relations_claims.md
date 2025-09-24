## UC04 — GraphCase: IE estruturada (entities, relations, claims) citando `unit_id`

Relacionado: `docs/usecases/uc04_extracao_grafo_ie_estruturada_entities_relations_claims.md`

### Nós e papéis
- **PrepareTextUnits**: Seleciona `unit_id` e texto base com metadados.
- **ExtractEntities**: Extrai entidades com tipos e spans.
- **ExtractRelations**: Extrai relações entre entidades com justificativas.
- **ExtractClaims**: Extrai afirmações (sujeito-predicado-objeto) com evidências.
- **PersistGraphBatch**: Persiste nós/arestas e citações para rastreabilidade.

### Arestas e condições
- `PrepareTextUnits → ExtractEntities`: quando `units.count > 0`.
- `ExtractEntities → ExtractRelations`: quando `entities.length > 0`.
- `ExtractEntities → ExtractClaims`: quando `entities.length > 0`.
- `ExtractRelations → PersistGraphBatch`: quando `relations.valid`.
- `ExtractClaims → PersistGraphBatch`: quando `claims.valid`.

### Schemas de nós

#### PrepareTextUnits
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sessionId": { "type": "string" },
    "units": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "unitId": { "type": "string" },
          "text": { "type": "string" },
          "metadata": { "type": "object" }
        },
        "required": ["unitId", "text"]
      }
    }
  },
  "required": ["sessionId", "units"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "units": { "type": "array", "items": { "type": "object" } }
  },
  "required": ["units"]
}
```

#### ExtractEntities
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "units": { "type": "array", "items": { "type": "object" } },
    "entityTypes": { "type": "array", "items": { "type": "string" } }
  },
  "required": ["units"]
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
      "items": {
        "type": "object",
        "properties": {
          "entityId": { "type": "string" },
          "type": { "type": "string" },
          "name": { "type": "string" },
          "unitId": { "type": "string" },
          "span": { "type": "array", "items": { "type": "number" } }
        },
        "required": ["entityId", "type", "name", "unitId"]
      }
    }
  },
  "required": ["entities"]
}
```

#### ExtractRelations
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "entities": { "type": "array", "items": { "type": "object" } }
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
    "relations": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "items": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "relationId": { "type": "string" },
              "type": { "type": "string" },
              "fromEntityId": { "type": "string" },
              "toEntityId": { "type": "string" },
              "unitId": { "type": "string" }
            },
            "required": ["relationId", "type", "fromEntityId", "toEntityId"]
          }
        }
      },
      "required": ["valid", "items"]
    }
  },
  "required": ["relations"]
}
```

#### ExtractClaims
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "entities": { "type": "array", "items": { "type": "object" } }
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
    "claims": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "items": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "claimId": { "type": "string" },
              "subjectId": { "type": "string" },
              "predicate": { "type": "string" },
              "object": { "type": "string" },
              "unitId": { "type": "string" }
            },
            "required": ["claimId", "subjectId", "predicate", "object"]
          }
        }
      },
      "required": ["valid", "items"]
    }
  },
  "required": ["claims"]
}
```

#### PersistGraphBatch
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "entities": { "type": "array", "items": { "type": "object" } },
    "relations": { "type": "object" },
    "claims": { "type": "object" }
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
    "persistResult": {
      "type": "object",
      "properties": {
        "persisted": { "type": "boolean" },
        "nodeCount": { "type": "number" },
        "edgeCount": { "type": "number" }
      },
      "required": ["persisted"]
    }
  },
  "required": ["persistResult"]
}
```


