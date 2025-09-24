## UC23 — GraphCase: Cache multicamadas

Relacionado: `docs/usecases/uc23_operacao_custos_cache_multicamadas.md`

### Nós e papéis
- **ResolveCacheStrategy**: Define política (local/process/mem/redis/vector/doc).
- **CheckCaches**: Verifica camadas em ordem e validade.
- **PopulateCaches**: Popula camadas faltantes com TTL/políticas.
- **ReturnCachedOrFresh**: Retorna do cache ou executa fonte e guarda.

### Arestas e condições
- `ResolveCacheStrategy → CheckCaches`: quando `strategy.valid`.
- `CheckCaches → ReturnCachedOrFresh`: quando `hit.found`.
- `CheckCaches → PopulateCaches`: quando `hit.found == false`.
- `PopulateCaches → ReturnCachedOrFresh`: quando `populate.valid`.

### Schemas de nós

#### ResolveCacheStrategy
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "policy": { "type": "string" } },
  "required": ["policy"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "strategy": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["strategy"]
}
```

#### CheckCaches
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "strategy": { "type": "object" }, "key": { "type": "string" } },
  "required": ["strategy", "key"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "hit": { "type": "object", "properties": { "found": { "type": "boolean" } }, "required": ["found"] } },
  "required": ["hit"]
}
```

#### PopulateCaches
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "strategy": { "type": "object" }, "value": { "type": "object" } },
  "required": ["strategy", "value"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "populate": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["populate"]
}
```

#### ReturnCachedOrFresh
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "hit": { "type": "object" }, "fallback": { "type": "object" } },
  "required": ["hit"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "result": { "type": "object", "properties": { "fromCache": { "type": "boolean" } }, "required": ["fromCache"] } },
  "required": ["result"]
}
```


