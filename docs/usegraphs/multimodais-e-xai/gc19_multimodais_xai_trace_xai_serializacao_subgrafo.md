## UC19 — GraphCase: Trace XAI: serialização de subgrafo/mermaid/Neo4j Bloom/GraphView

Relacionado: `docs/usecases/uc19_multimodais_xai_trace_xai_serializacao_subgrafo.md`

### Nós e papéis
- **StartTraceXAI**: Inicia captura de passos e artefatos por nó.
- **CaptureNodeSteps**: Registra prompts, respostas e métricas por nó.
- **SerializeSubgraph**: Serializa subgrafo em formatos (Mermaid/GraphView).
- **ExportVisualization**: Exporta para Bloom/arquivos/imagens.

### Arestas e condições
- `StartTraceXAI → CaptureNodeSteps`: quando `xai.started`.
- `CaptureNodeSteps → SerializeSubgraph`: quando `steps.count > 0`.
- `SerializeSubgraph → ExportVisualization`: quando `serialization.valid`.

### Schemas de nós

#### StartTraceXAI
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
  "properties": { "xai": { "type": "object", "properties": { "started": { "type": "boolean" } }, "required": ["started"] } },
  "required": ["xai"]
}
```

#### CaptureNodeSteps
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "xai": { "type": "object" } },
  "required": ["xai"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "steps": { "type": "object", "properties": { "count": { "type": "number" } }, "required": ["count"] }
  },
  "required": ["steps"]
}
```

#### SerializeSubgraph
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "steps": { "type": "object" }, "format": { "type": "string" } },
  "required": ["steps", "format"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "serialization": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["serialization"]
}
```

#### ExportVisualization
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "serialization": { "type": "object" } },
  "required": ["serialization"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "exportRef": { "type": "string" } },
  "required": ["exportRef"]
}
```


