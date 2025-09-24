## UC12 — GraphCase: DRIFT (global → local com reclassificação)

Relacionado: `docs/usecases/uc12_consulta_drift_hibrido_global_para_local.md`

### Nós e papéis
- **GlobalRetrieve**: Recupera contexto global (comunidades/QFS).
- **DetectDrift**: Detecta desvio entre intenção e contexto retornado.
- **ReclassifyToLocal**: Reclassifica intenção e busca contexto local.
- **ComposeHybridAnswer**: Combina global/local e compõe resposta.

### Arestas e condições
- `GlobalRetrieve → DetectDrift`: quando `globalContext.valid`.
- `DetectDrift → ReclassifyToLocal`: quando `drift.detected`.
- `ReclassifyToLocal → ComposeHybridAnswer`: quando `localContext.valid`.
- `DetectDrift → ComposeHybridAnswer`: quando `drift.detected == false`.

### Schemas de nós

#### GlobalRetrieve
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "string" }
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
    "globalContext": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" }
      },
      "required": ["valid"]
    }
  },
  "required": ["globalContext"]
}
```

#### DetectDrift
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "globalContext": { "type": "object" },
    "query": { "type": "string" }
  },
  "required": ["globalContext", "query"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "drift": {
      "type": "object",
      "properties": {
        "detected": { "type": "boolean" },
        "score": { "type": "number" }
      },
      "required": ["detected"]
    }
  },
  "required": ["drift"]
}
```

#### ReclassifyToLocal
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "query": { "type": "string" }
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
    "localContext": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" }
      },
      "required": ["valid"]
    }
  },
  "required": ["localContext"]
}
```

#### ComposeHybridAnswer
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "globalContext": { "type": "object" },
    "drift": { "type": "object" },
    "localContext": { "type": "object" }
  }
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


