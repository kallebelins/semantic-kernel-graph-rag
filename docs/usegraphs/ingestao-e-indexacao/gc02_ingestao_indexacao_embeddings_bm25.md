## UC02 — GraphCase: Embeddings + BM25 (filtros por tags/setor/tema/lang)

Relacionado: `docs/usecases/uc02_ingestao_indexacao_embeddings_bm25.md`

### Nós e papéis
- **PrepareIndexInputs**: Recebe chunks e filtros de escopo (tags/setor/tema/lang).
- **GenerateEmbeddings**: Gera embeddings para cada chunk selecionado.
- **BuildBM25**: Constrói índice BM25 para o mesmo conjunto.
- **PersistIndex**: Persiste índice vetorial e BM25 com metadados e versões.

### Arestas e condições
- `PrepareIndexInputs → GenerateEmbeddings`: quando `selection.count > 0`.
- `PrepareIndexInputs → BuildBM25`: quando `selection.count > 0`.
- `GenerateEmbeddings → PersistIndex`: quando `vectors.valid`.
- `BuildBM25 → PersistIndex`: quando `bm25.valid`.

### Schemas de nós

#### PrepareIndexInputs
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sessionId": { "type": "string" },
    "chunks": { "type": "array", "items": { "type": "object" } },
    "filters": {
      "type": "object",
      "properties": {
        "tags": { "type": "array", "items": { "type": "string" } },
        "sector": { "type": "string" },
        "theme": { "type": "string" },
        "lang": { "type": "string" }
      }
    }
  },
  "required": ["sessionId", "chunks"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "selection": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "chunkIds": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["count", "chunkIds"]
    }
  },
  "required": ["selection"]
}
```

#### GenerateEmbeddings
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "selection": { "type": "object" },
    "embeddingModel": { "type": "string" }
  },
  "required": ["selection", "embeddingModel"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "vectors": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "items": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "chunkId": { "type": "string" },
              "vector": { "type": "array", "items": { "type": "number" } }
            },
            "required": ["chunkId", "vector"]
          }
        }
      },
      "required": ["valid", "items"]
    }
  },
  "required": ["vectors"]
}
```

#### BuildBM25
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "selection": { "type": "object" },
    "tokenizer": { "type": "string" }
  },
  "required": ["selection"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "bm25": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "docCount": { "type": "number" },
        "indexRef": { "type": "string" }
      },
      "required": ["valid", "docCount"]
    }
  },
  "required": ["bm25"]
}
```

#### PersistIndex
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "vectors": { "type": "object" },
    "bm25": { "type": "object" },
    "version": { "type": "string" }
  },
  "required": ["vectors", "bm25"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "persistedIndex": {
      "type": "object",
      "properties": {
        "persisted": { "type": "boolean" },
        "vectorIndexRef": { "type": "string" },
        "bm25IndexRef": { "type": "string" }
      },
      "required": ["persisted"]
    }
  },
  "required": ["persistedIndex"]
}
```


