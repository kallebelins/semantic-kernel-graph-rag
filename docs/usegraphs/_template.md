### Padrão de saída (GraphCase)

- **Estrutura e ordem**
  - Título: `## UCXX — GraphCase: <título do caso>`
  - Relacionado: path do UC correspondente em `docs/usecases`
  - Seções:
    - `### Nós e papéis`: lista de nós com função resumida
    - `### Arestas e condições`: `NodeA → NodeB` com condição/guarda
    - `### Schemas de nós`: para cada nó, `Input schema` e `Output schema` em JSON Schema 2020-12

- **Convenções**
  - **Nome de nó**: PascalCase (ex.: `NormalizeEvent`, `HybridRetrieve`)
  - **Campos**: camelCase (ex.: `sessionId`, `messageExternalId`, `topK`)
  - **JSON Schema**: usar `$schema` 2020-12; `type: "object"` na raiz; `required` sempre presente
  - **Arestas**: formato literal com seta `→` e condição iniciada por “quando”
  - **Tipos comuns**:
    - IDs: `string`
    - Flags: `boolean` (ex.: `persisted`, `valid`, `sealed`, `assigned`)
    - Listas: `array` com `items`
    - Datas: `string` com `format: "date-time"`
    - Enums quando aplicável (ex.: `route`, `type`, `finalStatus`)

### Template para novas criações

```markdown
## UCXX — GraphCase: <título do caso>

Relacionado: `docs/usecases/ucXX_<categoria>_<nomeusecase>.md`

### Nós e papéis
- **<Node1>**: <papel do nó em 1 linha>.
- **<Node2>**: <papel do nó>.
- ...

### Arestas e condições
- `<Node1> → <Node2>`: quando <condição objetiva>.
- `<Node2> → <Node3>`: quando <condição>.
- ...

### Schemas de nós

#### <Node1>
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "<campoEntrada1>": { "type": "string" },
    "<objEntradaOpcional>": { "type": "object" }
  },
  "required": ["<campoEntrada1>"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "<objSaida>": {
      "type": "object",
      "properties": {
        "<campoSaida1>": { "type": "string" },
        "<flagSaida>": { "type": "boolean" }
      },
      "required": ["<campoSaida1>"]
    }
  },
  "required": ["<objSaida>"]
}
```

#### <Node2>
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "<dependenciaDeNode1>": { "type": "object" }
  },
  "required": ["<dependenciaDeNode1>"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "<resultadoNode2>": { "type": "array", "items": { "type": "object" } }
  },
  "required": ["<resultadoNode2>"]
}
```

<!-- Repita a subseção de schemas para todos os nós -->
```

- **Sugestão de nomes recorrentes**: `session`, `sessionId`, `message`, `messageId`, `messageExternalId`, `instruction`, `instructionId`, `version`, `route` (ai|human), `retrieved`, `context`, `prompt`, `answer`, `citations`, `valid`, `assigned`, `sealed`, `finalStatus`, `persisted`.

- **Critério de qualidade**: cada nó deve ter `Input schema` e `Output schema` completos e coerentes com seus pares na seção de arestas; todo campo consumido por um nó deve aparecer como `required` em seu input.

- Caso deseje, posso gerar um “skeleton” automaticamente para um novo UC a partir desse padrão.