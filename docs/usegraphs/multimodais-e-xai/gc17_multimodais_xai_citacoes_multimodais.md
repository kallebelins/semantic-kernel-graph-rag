## UC17 — GraphCase: Citações multimodais: `time_range`, `bbox`, `page/section/heading`

Relacionado: `docs/usecases/uc17_multimodais_xai_citacoes_multimodais.md`

### Nós e papéis
- **ParseCitationsRequest**: Recebe pedido de citações multimodais.
- **ExtractCitations**: Extrai `time_range`, `bbox`, `page/section/heading` por mídia.
- **ValidateCitations**: Valida referências/marcadores.
- **PersistCitations**: Persiste citações com referência ao conteúdo.

### Arestas e condições
- `ParseCitationsRequest → ExtractCitations`: quando `request.valid`.
- `ExtractCitations → ValidateCitations`: quando `citations.count > 0`.
- `ValidateCitations → PersistCitations`: quando `citations.valid`.

### Schemas de nós

#### ParseCitationsRequest
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "request": { "type": "object" } },
  "required": ["request"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "request": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["request"]
}
```

#### ExtractCitations
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "request": { "type": "object" } },
  "required": ["request"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "citations": {
      "type": "object",
      "properties": {
        "count": { "type": "number" },
        "items": { "type": "array", "items": { "type": "object" } }
      },
      "required": ["count", "items"]
    }
  },
  "required": ["citations"]
}
```

#### ValidateCitations
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "citations": { "type": "object" } },
  "required": ["citations"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "citations": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["citations"]
}
```

#### PersistCitations
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "citations": { "type": "object" } },
  "required": ["citations"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "persisted": { "type": "boolean" } },
  "required": ["persisted"]
}
```


