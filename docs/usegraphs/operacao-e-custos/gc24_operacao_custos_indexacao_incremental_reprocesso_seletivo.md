## UC24 — GraphCase: Indexação incremental e reprocesso seletivo

Relacionado: `docs/usecases/uc24_operacao_custos_indexacao_incremental_reprocesso_seletivo.md`

### Nós e papéis
- **DetectChanges**: Detecta alterações/novos documentos.
- **BuildIncrementalIndex**: Constrói índices incrementais.
- **SelectiveReprocess**: Reprocessa apenas partes afetadas.
- **PersistIncremental**: Persiste resultados e avança ponteiros.

### Arestas e condições
- `DetectChanges → BuildIncrementalIndex`: quando `changes.count > 0`.
- `BuildIncrementalIndex → SelectiveReprocess`: quando `incremental.valid`.
- `SelectiveReprocess → PersistIncremental`: quando `reprocess.valid`.

### Schemas de nós

#### DetectChanges
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "since": { "type": "string", "format": "date-time" } }
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "changes": { "type": "object", "properties": { "count": { "type": "number" } }, "required": ["count"] } },
  "required": ["changes"]
}
```

#### BuildIncrementalIndex
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "changes": { "type": "object" } },
  "required": ["changes"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "incremental": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["incremental"]
}
```

#### SelectiveReprocess
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "incremental": { "type": "object" } },
  "required": ["incremental"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "reprocess": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["reprocess"]
}
```

#### PersistIncremental
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "incremental": { "type": "object" }, "reprocess": { "type": "object" } },
  "required": ["incremental", "reprocess"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "persistResult": { "type": "object", "properties": { "persisted": { "type": "boolean" } }, "required": ["persisted"] } },
  "required": ["persistResult"]
}
```


