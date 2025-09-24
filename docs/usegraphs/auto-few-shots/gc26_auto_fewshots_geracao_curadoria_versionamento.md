## UC26 — GraphCase: Geração, curadoria e versionamento por projeto/idioma

Relacionado: `docs/usecases/uc26_auto_fewshots_geracao_curadoria_versionamento.md`

### Nós e papéis
- **GenerateFewShots**: Gera exemplos por tarefa/projeto/idioma.
- **CurateFewShots**: Seleciona e ajusta exemplos (qualidade/diversidade).
- **VersionFewShots**: Atribui versão/labels e mudanças.
- **PersistFewShots**: Persiste e indexa por projeto/idioma.

### Arestas e condições
- `GenerateFewShots → CurateFewShots`: quando `generated.count > 0`.
- `CurateFewShots → VersionFewShots`: quando `curated.valid`.
- `VersionFewShots → PersistFewShots`: quando `versioned.valid`.

### Schemas de nós

#### GenerateFewShots
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "task": { "type": "string" }, "projectId": { "type": "string" }, "lang": { "type": "string" } },
  "required": ["task", "projectId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "generated": { "type": "object", "properties": { "count": { "type": "number" } }, "required": ["count"] } },
  "required": ["generated"]
}
```

#### CurateFewShots
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "generated": { "type": "object" } },
  "required": ["generated"]
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

#### VersionFewShots
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "curated": { "type": "object" }, "version": { "type": "string" } },
  "required": ["curated"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "versioned": { "type": "object", "properties": { "valid": { "type": "boolean" }, "version": { "type": "string" } }, "required": ["valid", "version"] } },
  "required": ["versioned"]
}
```

#### PersistFewShots
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "versioned": { "type": "object" }, "projectId": { "type": "string" }, "lang": { "type": "string" } },
  "required": ["versioned", "projectId"]
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


