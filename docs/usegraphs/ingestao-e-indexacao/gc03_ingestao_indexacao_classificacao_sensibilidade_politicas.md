## UC03 — GraphCase: Classificação de sensibilidade e políticas de privacidade

Relacionado: `docs/usecases/uc03_ingestao_indexacao_classificacao_sensibilidade_politicas.md`

### Nós e papéis
- **PrepareDocuments**: Seleciona documentos/chunks e contexto de compliance.
- **ClassifySensitivity**: Classifica sensibilidade (ex.: PII/PHI/PCI) e confidencialidade.
- **ApplyPolicies**: Determina políticas aplicáveis por regra/região/propósito.
- **RedactOrMask**: Redige/mascara campos sensíveis conforme política.
- **PersistPolicyTags**: Persiste labels, justificativas e trilhas de decisão.

### Arestas e condições
- `PrepareDocuments → ClassifySensitivity`: quando `selection.count > 0`.
- `ClassifySensitivity → ApplyPolicies`: quando `sensitivity.valid`.
- `ApplyPolicies → RedactOrMask`: quando `policies.assigned`.
- `RedactOrMask → PersistPolicyTags`: quando `transformed.valid`.

### Schemas de nós

#### PrepareDocuments
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sessionId": { "type": "string" },
    "chunks": { "type": "array", "items": { "type": "object" } },
    "complianceContext": {
      "type": "object",
      "properties": {
        "region": { "type": "string" },
        "purpose": { "type": "string" }
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

#### ClassifySensitivity
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "selection": { "type": "object" },
    "classifiers": { "type": "array", "items": { "type": "string" } }
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
    "sensitivity": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "labels": { "type": "array", "items": { "type": "string" } },
        "explanations": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["valid", "labels"]
    }
  },
  "required": ["sensitivity"]
}
```

#### ApplyPolicies
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sensitivity": { "type": "object" },
    "complianceContext": { "type": "object" }
  },
  "required": ["sensitivity"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "policies": {
      "type": "object",
      "properties": {
        "assigned": { "type": "boolean" },
        "rules": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["assigned"]
    }
  },
  "required": ["policies"]
}
```

#### RedactOrMask
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "selection": { "type": "object" },
    "policies": { "type": "object" }
  },
  "required": ["selection", "policies"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "transformed": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "chunkIds": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["valid", "chunkIds"]
    }
  },
  "required": ["transformed"]
}
```

#### PersistPolicyTags
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sensitivity": { "type": "object" },
    "policies": { "type": "object" },
    "transformed": { "type": "object" }
  },
  "required": ["sensitivity", "policies", "transformed"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "persisted": {
      "type": "object",
      "properties": {
        "persisted": { "type": "boolean" },
        "labelCount": { "type": "number" }
      },
      "required": ["persisted"]
    }
  },
  "required": ["persisted"]
}
```


