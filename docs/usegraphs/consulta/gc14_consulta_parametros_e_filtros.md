## UC14 — GraphCase: Parâmetros e filtros

Relacionado: `docs/usecases/uc14_consulta_parametros_e_filtros.md`

### Nós e papéis
- **ParseParameters**: Lê parâmetros e valida tipos/intervalos.
- **ApplyFilters**: Aplica filtros em tempo de consulta.
- **ValidateConstraints**: Garante consistência e limites operacionais.

### Arestas e condições
- `ParseParameters → ApplyFilters`: quando `params.valid`.
- `ApplyFilters → ValidateConstraints`: quando `filters.applied`.

### Schemas de nós

#### ParseParameters
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "params": { "type": "object" } },
  "required": ["params"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "params": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" }
      },
      "required": ["valid"]
    }
  },
  "required": ["params"]
}
```

#### ApplyFilters
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "params": { "type": "object" },
    "query": { "type": "string" }
  },
  "required": ["params"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "filters": {
      "type": "object",
      "properties": {
        "applied": { "type": "boolean" }
      },
      "required": ["applied"]
    }
  },
  "required": ["filters"]
}
```

#### ValidateConstraints
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "filters": { "type": "object" } },
  "required": ["filters"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "constraints": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" }
      },
      "required": ["valid"]
    }
  },
  "required": ["constraints"]
}
```


