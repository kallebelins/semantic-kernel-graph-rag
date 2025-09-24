## UC18 — GraphCase: Consultas cross-modais e ordenação unificada

Relacionado: `docs/usecases/uc18_multimodais_xai_consultas_cross_modais_ordenacao_unificada.md`

### Nós e papéis
- **NormalizeCrossModalQuery**: Normaliza consulta com alvos modais.
- **RetrievePerModality**: Recupera resultados por modalidade (texto/imagem/áudio/vídeo).
- **UnifiedRank**: Normaliza scores e unifica ordenação.
- **ComposeCrossModalAnswer**: Compõe resposta com itens heterogêneos.

### Arestas e condições
- `NormalizeCrossModalQuery → RetrievePerModality`: quando `query.valid`.
- `RetrievePerModality → UnifiedRank`: quando `results.total > 0`.
- `UnifiedRank → ComposeCrossModalAnswer`: quando `ranking.valid`.

### Schemas de nós

#### NormalizeCrossModalQuery
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "query": { "type": "string" }, "modalities": { "type": "array", "items": { "type": "string" } } },
  "required": ["query"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "query": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["query"]
}
```

#### RetrievePerModality
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "query": { "type": "object" } },
  "required": ["query"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "results": {
      "type": "object",
      "properties": { "total": { "type": "number" }, "buckets": { "type": "object" } },
      "required": ["total", "buckets"]
    }
  },
  "required": ["results"]
}
```

#### UnifiedRank
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "results": { "type": "object" } },
  "required": ["results"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "ranking": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["ranking"]
}
```

#### ComposeCrossModalAnswer
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "ranking": { "type": "object" } },
  "required": ["ranking"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "answer": { "type": "object", "properties": { "text": { "type": "string" } }, "required": ["text"] } },
  "required": ["answer"]
}
```


