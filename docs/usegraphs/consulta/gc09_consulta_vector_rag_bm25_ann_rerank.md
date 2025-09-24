## UC09 — GraphCase: Vector RAG (BM25 ∪ ANN com re-rank)

Relacionado: `docs/usecases/uc09_consulta_vector_rag_bm25_ann_rerank.md`

### Nós e papéis
- **ParseQuery**: Normaliza consulta e extrai filtros.
- **HybridRetrieve**: Recupera candidatos via BM25 ∪ ANN.
- **RerankResults**: Reordena candidatos por modelo de re-rank.
- **BuildAnswer**: Monta resposta com citações.

### Arestas e condições
- `ParseQuery → HybridRetrieve`: quando `query.valid`.
- `HybridRetrieve → RerankResults`: quando `candidates.count > 0`.
- `RerankResults → BuildAnswer`: quando `reranked.count > 0`.

### Schemas de nós

#### ParseQuery
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "string" },
    "filters": { "type": "object" }
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
    "query": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "text": { "type": "string" },
        "filters": { "type": "object" }
      },
      "required": ["valid", "text"]
    }
  },
  "required": ["query"]
}
```

#### HybridRetrieve
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "object" },
    "topK": { "type": "number" }
  },
  "required": ["query", "topK"]
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
        "items": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["count", "items"]
    }
  },
  "required": ["candidates"]
}
```

#### RerankResults
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "candidates": { "type": "object" },
    "model": { "type": "string" }
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
    "reranked": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "items": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["count", "items"]
    }
  },
  "required": ["reranked"]
}
```

#### BuildAnswer
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "reranked": { "type": "object" },
    "query": { "type": "object" }
  },
  "required": ["reranked", "query"]
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


