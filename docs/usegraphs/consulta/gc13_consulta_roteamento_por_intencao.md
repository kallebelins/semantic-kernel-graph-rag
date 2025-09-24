## UC13 — GraphCase: Roteamento por intenção (heurísticas e/ou classificadores)

Relacionado: `docs/usecases/uc13_consulta_roteamento_por_intencao.md`

### Nós e papéis
- **ParseIntent**: Classifica intenção (ex.: search, graph, tools, chat).
- **RouteQuery**: Seleciona rota e parâmetros.
- **ExecuteRoute**: Executa rota escolhida (subgrafo reutilizável).
- **ComposeRoutedAnswer**: Consolida resultado e metadados de rota.

### Arestas e condições
- `ParseIntent → RouteQuery`: quando `intent.valid`.
- `RouteQuery → ExecuteRoute`: quando `route.assigned`.
- `ExecuteRoute → ComposeRoutedAnswer`: quando `result.valid`.

### Schemas de nós

#### ParseIntent
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "query": { "type": "string" } },
  "required": ["query"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "intent": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "route": { "type": "string" }
      },
      "required": ["valid", "route"]
    }
  },
  "required": ["intent"]
}
```

#### RouteQuery
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "intent": { "type": "object" } },
  "required": ["intent"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "route": {
      "type": "object",
      "properties": {
        "assigned": { "type": "boolean" },
        "name": { "type": "string" }
      },
      "required": ["assigned", "name"]
    }
  },
  "required": ["route"]
}
```

#### ExecuteRoute
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "route": { "type": "object" } },
  "required": ["route"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "result": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "payload": { "type": "object" }
      },
      "required": ["valid"]
    }
  },
  "required": ["result"]
}
```

#### ComposeRoutedAnswer
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "result": { "type": "object" },
    "route": { "type": "object" }
  },
  "required": ["result", "route"]
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
        "route": { "type": "string" }
      },
      "required": ["text", "route"]
    }
  },
  "required": ["answer"]
}
```


