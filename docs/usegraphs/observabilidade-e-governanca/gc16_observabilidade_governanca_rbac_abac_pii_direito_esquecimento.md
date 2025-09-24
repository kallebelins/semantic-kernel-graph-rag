## UC16 — GraphCase: RBAC/ABAC, redação/mascaramento de PII, direito ao esquecimento

Relacionado: `docs/usecases/uc16_observabilidade_governanca_rbac_abac_pii_direito_esquecimento.md`

### Nós e papéis
- **ResolveSubjectContext**: Resolve identidade, papéis e atributos (RBAC/ABAC).
- **AuthorizeAccess**: Avalia políticas e decide acesso por recurso.
- **ApplyRedaction**: Redige/mascara campos sensíveis conforme decisão.
- **ProcessErasureRequest**: Processa solicitações de esquecimento (soft/hard delete).
- **PersistAudit**: Registra auditoria de decisões e ações.

### Arestas e condições
- `ResolveSubjectContext → AuthorizeAccess`: quando `subject.valid`.
- `AuthorizeAccess → ApplyRedaction`: quando `decision.permit` e `obligations.contains("redact")`.
- `AuthorizeAccess → PersistAudit`: quando `decision.finalized`.
- `ProcessErasureRequest → PersistAudit`: quando `erasure.applied`.

### Schemas de nós

#### ResolveSubjectContext
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "subjectId": { "type": "string" } },
  "required": ["subjectId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "subject": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "roles": { "type": "array", "items": { "type": "string" } },
        "attributes": { "type": "object" }
      },
      "required": ["valid"]
    }
  },
  "required": ["subject"]
}
```

#### AuthorizeAccess
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "subject": { "type": "object" },
    "resource": { "type": "string" },
    "action": { "type": "string" }
  },
  "required": ["subject", "resource", "action"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "decision": {
      "type": "object",
      "properties": {
        "finalized": { "type": "boolean" },
        "permit": { "type": "boolean" },
        "obligations": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["finalized", "permit"]
    }
  },
  "required": ["decision"]
}
```

#### ApplyRedaction
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "decision": { "type": "object" }, "data": { "type": "object" } },
  "required": ["decision", "data"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "redacted": { "type": "object" }
  },
  "required": ["redacted"]
}
```

#### ProcessErasureRequest
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "subjectId": { "type": "string" }, "resource": { "type": "string" } },
  "required": ["subjectId", "resource"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "erasure": {
      "type": "object",
      "properties": { "applied": { "type": "boolean" } },
      "required": ["applied"]
    }
  },
  "required": ["erasure"]
}
```

#### PersistAudit
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "decision": { "type": "object" } },
  "required": ["decision"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "auditRef": { "type": "string" }
  },
  "required": ["auditRef"]
}
```


