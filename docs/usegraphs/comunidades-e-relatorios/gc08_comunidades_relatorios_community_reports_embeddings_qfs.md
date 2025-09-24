## UC08 — GraphCase: Community Reports e embeddings para QFS

Relacionado: `docs/usecases/uc08_comunidades_relatorios_community_reports_embeddings_qfs.md`

### Nós e papéis
- **SelectCommunity**: Seleciona comunidade/nível e recorte.
- **SummarizeCommunity**: Gera relatório textual com estatísticas e destaques.
- **EmbedReports**: Gera embeddings do relatório para QFS.
- **PersistReports**: Persiste relatório, embeddings e metadados.

### Arestas e condições
- `SelectCommunity → SummarizeCommunity`: quando `community.valid`.
- `SummarizeCommunity → EmbedReports`: quando `report.valid`.
- `EmbedReports → PersistReports`: quando `embeddings.valid`.

### Schemas de nós

#### SelectCommunity
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "communityId": { "type": "string" },
    "level": { "type": "number" }
  },
  "required": ["communityId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "community": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" }
      },
      "required": ["valid"]
    }
  },
  "required": ["community"]
}
```

#### SummarizeCommunity
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "community": { "type": "object" }
  },
  "required": ["community"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "report": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "text": { "type": "string" }
      },
      "required": ["valid", "text"]
    }
  },
  "required": ["report"]
}
```

#### EmbedReports
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "report": { "type": "object" },
    "embeddingModel": { "type": "string" }
  },
  "required": ["report"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "embeddings": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "vector": { "type": "array", "items": { "type": "number" } }
      },
      "required": ["valid", "vector"]
    }
  },
  "required": ["embeddings"]
}
```

#### PersistReports
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "community": { "type": "object" },
    "report": { "type": "object" },
    "embeddings": { "type": "object" }
  },
  "required": ["community", "report", "embeddings"]
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
        "reportId": { "type": "string" }
      },
      "required": ["persisted"]
    }
  },
  "required": ["persistResult"]
}
```


