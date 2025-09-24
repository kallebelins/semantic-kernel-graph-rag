## UC25 — GraphCase: Pessoas/Equipes/Projetos e relações com conteúdos

Relacionado: `docs/usecases/uc25_integracao_social_organizacional_pessoas_equipes_projetos_relacoes_conteudos.md`

### Nós e papéis
- **IngestOrgEntities**: Ingesta pessoas/equipes/projetos com atributos.
- **LinkPeopleProjects**: Cria relações pessoa↔projeto/equipe.
- **LinkContents**: Associa conteúdos a entidades (autoria/pertencimento).
- **PersistOrgGraph**: Persiste entidades/relações e versões.

### Arestas e condições
- `IngestOrgEntities → LinkPeopleProjects`: quando `entities.valid`.
- `LinkPeopleProjects → LinkContents`: quando `links.valid`.
- `LinkContents → PersistOrgGraph`: quando `contentLinks.valid`.

### Schemas de nós

#### IngestOrgEntities
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "batch": { "type": "array", "items": { "type": "object" } } },
  "required": ["batch"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "entities": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["entities"]
}
```

#### LinkPeopleProjects
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "entities": { "type": "object" } },
  "required": ["entities"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "links": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["links"]
}
```

#### LinkContents
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "links": { "type": "object" }, "contents": { "type": "array", "items": { "type": "object" } } },
  "required": ["links"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "contentLinks": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["contentLinks"]
}
```

#### PersistOrgGraph
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "entities": { "type": "object" }, "contentLinks": { "type": "object" } },
  "required": ["entities", "contentLinks"]
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


