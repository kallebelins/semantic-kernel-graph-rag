## UC21 — GraphCase: Exportações para Data Lake/BI e dashboards

Relacionado: `docs/usecases/uc21_insights_bi_exportacoes_datalake_bi_dashboards.md`

### Nós e papéis
- **SelectDatasets**: Seleciona datasets, schemas e partições.
- **ExportToDataLake**: Exporta em formatos (Parquet/Delta/CSV) com catálogo.
- **PublishBIDashboards**: Publica/executa refresh em BI/dashboards.

### Arestas e condições
- `SelectDatasets → ExportToDataLake`: quando `datasets.count > 0`.
- `ExportToDataLake → PublishBIDashboards`: quando `export.valid`.

### Schemas de nós

#### SelectDatasets
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "datasets": { "type": "array", "items": { "type": "string" } } },
  "required": ["datasets"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "selection": { "type": "object", "properties": { "count": { "type": "number" } }, "required": ["count"] } },
  "required": ["selection"]
}
```

#### ExportToDataLake
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "selection": { "type": "object" }, "format": { "type": "string" } },
  "required": ["selection", "format"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "export": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["export"]
}
```

#### PublishBIDashboards
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "export": { "type": "object" } },
  "required": ["export"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "bi": { "type": "object", "properties": { "published": { "type": "boolean" } }, "required": ["published"] } },
  "required": ["bi"]
}
```


