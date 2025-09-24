## UC15 — GraphCase: Traços por nó (latência, custo, tokens, provider/model)

Relacionado: `docs/usecases/uc15_observabilidade_governanca_tracos_por_no.md`

### Nós e papéis
- **StartTrace**: Inicia sessão de trace por chamada.
- **RecordNodeMetrics**: Registra métricas por nó (latência/custo/tokens/provider/model).
- **AggregateTrace**: Agrega métricas e calcula totais.
- **PersistTrace**: Persiste trace e expõe referência.

### Arestas e condições
- `StartTrace → RecordNodeMetrics`: quando `trace.started`.
- `RecordNodeMetrics → AggregateTrace`: quando `metrics.count > 0`.
- `AggregateTrace → PersistTrace`: quando `aggregate.valid`.

### Schemas de nós

#### StartTrace
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "sessionId": { "type": "string" } },
  "required": ["sessionId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "trace": {
      "type": "object",
      "properties": { "started": { "type": "boolean" } },
      "required": ["started"]
    }
  },
  "required": ["trace"]
}
```

#### RecordNodeMetrics
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "trace": { "type": "object" },
    "nodeId": { "type": "string" },
    "metrics": {
      "type": "object",
      "properties": {
        "latencyMs": { "type": "number" },
        "cost": { "type": "number" },
        "tokens": { "type": "number" },
        "provider": { "type": "string" },
        "model": { "type": "string" }
      }
    }
  },
  "required": ["trace", "nodeId", "metrics"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "metrics": {
      "type": "object",
      "properties": { "count": { "type": "number" } },
      "required": ["count"]
    }
  },
  "required": ["metrics"]
}
```

#### AggregateTrace
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "metrics": { "type": "object" } },
  "required": ["metrics"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "aggregate": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "totalCost": { "type": "number" },
        "totalTokens": { "type": "number" }
      },
      "required": ["valid"]
    }
  },
  "required": ["aggregate"]
}
```

#### PersistTrace
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "aggregate": { "type": "object" } },
  "required": ["aggregate"]
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
      "properties": { "persisted": { "type": "boolean" } },
      "required": ["persisted"]
    }
  },
  "required": ["persistResult"]
}
```


