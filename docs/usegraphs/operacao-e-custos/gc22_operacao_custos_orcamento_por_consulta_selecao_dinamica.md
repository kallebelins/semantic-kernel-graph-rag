## UC22 — GraphCase: Orçamento por consulta e seleção dinâmica de contexto

Relacionado: `docs/usecases/uc22_operacao_custos_orcamento_por_consulta_selecao_dinamica.md`

### Nós e papéis
- **InitBudget**: Inicializa orçamento/limites por consulta.
- **EstimateCosts**: Estima custo por rota/estratégia.
- **SelectContextDynamically**: Seleciona contexto ótimo sob restrição de orçamento.
- **EnforceBudget**: Garante execução dentro do orçamento.

### Arestas e condições
- `InitBudget → EstimateCosts`: quando `budget.valid`.
- `EstimateCosts → SelectContextDynamically`: quando `estimates.valid`.
- `SelectContextDynamically → EnforceBudget`: quando `selection.valid`.

### Schemas de nós

#### InitBudget
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "maxCost": { "type": "number" }, "maxTokens": { "type": "number" } },
  "required": ["maxCost"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "budget": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["budget"]
}
```

#### EstimateCosts
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "budget": { "type": "object" }, "routes": { "type": "array", "items": { "type": "string" } } },
  "required": ["budget"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "estimates": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["estimates"]
}
```

#### SelectContextDynamically
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "estimates": { "type": "object" } },
  "required": ["estimates"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "selection": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["selection"]
}
```

#### EnforceBudget
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "selection": { "type": "object" }, "budget": { "type": "object" } },
  "required": ["selection", "budget"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": { "enforced": { "type": "object", "properties": { "valid": { "type": "boolean" } }, "required": ["valid"] } },
  "required": ["enforced"]
}
```


