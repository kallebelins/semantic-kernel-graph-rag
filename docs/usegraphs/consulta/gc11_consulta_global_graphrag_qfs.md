## UC11 — GraphCase: Global GraphRAG (QFS por comunidades)

Relacionado: `docs/usecases/uc11_consulta_global_graphrag_qfs.md`

### Nós e papéis
- **SelectCommunitiesForQuery**: Seleciona comunidades relevantes para a consulta.
- **GenerateQFS**: Gera QFS (query-focused summaries) por comunidade.
- **RetrieveTopReports**: Seleciona relatórios/trechos por score.
- **ComposeGlobalAnswer**: Monta resposta global com citações.

### Arestas e condições
- `SelectCommunitiesForQuery → GenerateQFS`: quando `communities.count > 0`.
- `GenerateQFS → RetrieveTopReports`: quando `summaries.valid`.
- `RetrieveTopReports → ComposeGlobalAnswer`: quando `reports.count > 0`.

### Schemas de nós

#### SelectCommunitiesForQuery
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "string" },
    "topK": { "type": "number" }
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
    "communities": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "ids": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["count", "ids"]
    }
  },
  "required": ["communities"]
}
```

#### GenerateQFS
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "communities": { "type": "object" },
    "qfsModel": { "type": "string" }
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
    "summaries": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "items": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["valid", "items"]
    }
  },
  "required": ["summaries"]
}
```

#### RetrieveTopReports
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "summaries": { "type": "object" },
    "k": { "type": "number" }
  },
  "required": ["summaries", "k"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "reports": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "items": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["count", "items"]
    }
  },
  "required": ["reports"]
}
```

#### ComposeGlobalAnswer
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "reports": { "type": "object" }
  },
  "required": ["reports"]
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


