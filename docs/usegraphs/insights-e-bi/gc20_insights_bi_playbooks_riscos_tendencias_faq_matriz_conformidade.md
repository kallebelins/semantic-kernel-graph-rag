## UC20 — GraphCase: Playbooks: riscos, tendências, FAQ, matriz de conformidade

Relacionado: `docs/usecases/uc20_insights_bi_playbooks_riscos_tendencias_faq_matriz_conformidade.md`

### Nós e papéis
- **IdentifyPlaybookScope**: Define escopo (riscos/tendências/FAQ/conformidade).
- **GenerateInsights**: Gera insights por fonte/comunidade/período.
- **CurateAndTag**: Cura, deduplica e tagueia conteúdos.
- **PersistPlaybook**: Persiste playbook e versões.

### Arestas e condições
- `IdentifyPlaybookScope → GenerateInsights`: quando `scope.valid`.
- `GenerateInsights → CurateAndTag`: quando `insights.count > 0`.
- `CurateAndTag → PersistPlaybook`: quando `curated.valid`.

### Schemas de nós

#### IdentifyPlaybookScope
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "scope": { "type": "string" } },
  "required": ["scope"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "scope": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["scope"]
}
```

#### GenerateInsights
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "scope": { "type": "object" } },
  "required": ["scope"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "insights": { "type": "object", "properties": { "count": { "type": "number" } }, "required": ["count"] } },
  "required": ["insights"]
}
```

#### CurateAndTag
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "insights": { "type": "object" } },
  "required": ["insights"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "curated": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["curated"]
}
```

#### PersistPlaybook
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "curated": { "type": "object" } },
  "required": ["curated"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "playbookRef": { "type": "string" } },
  "required": ["playbookRef"]
}
```


