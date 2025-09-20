# Usecase: IE estruturada (entities, relations, claims) citando `unit_id`

## Contexto e Problema
Após a fragmentação, é necessário extrair estruturas de conhecimento de forma padronizada e citável. A IE deve produzir entidades, relações e afirmações com referência explícita às unidades de evidência (`unit_id`) para permitir transparência, auditoria e consolidação no grafo do projeto.

## Objetivo
Gerar saída normalizada de IE com alta precisão/recall e manter citações de evidência por `TextUnit`. A saída será usada para fusão/merge no grafo (evitando duplicatas via aliases e chaves de bloqueio) e consultas locais/globais.

## Atores e Escopo
- Usuários: Cientista/Arquiteto de Conhecimento; Operador de Ingestão
- Serviços: `IEExtractor`, `GraphMerge` (consome a saída), Observabilidade
- Escopo: Extração estruturada por unidade; não cobre merge nem comunidades

## Pré-condições
- `TextUnit[]` persistidos e acessíveis por projeto
- Perfis de prompt e few-shots para tarefa de extração definidos no projeto
- Políticas de privacidade e sensibilidade carregadas para governança

## Entradas
- `TextUnit[]` com metadados: `{unit_id, doc_id, page, section, heading, tags, setor, tema, lang}`
- `PromptProfile` de extração (com exemplos) e parâmetros do provider/modelo

## Fluxo (alto nível)
- Selecionar lote de `TextUnit` elegíveis (filtros e orçamento)
- IEExtractor.Run → saída JSON normalizada `{entities[], relations[], claims[]}` com citações `unit_id`
- Pós-processo: normalização de texto, aliases, chaves de bloqueio, thresholds de confiança
- Persistir artefato de IE por unidade (para auditoria) e encaminhar para `GraphMerge`
- Nós SKGraph: `IEExtractorNode`, `FunctionGraphNode(PostProcessIE)`, `ResultAggregator` (opcional)

## Regras/Políticas e Guardrails
- Citação obrigatória: cada `entity mention`, `relation evidence` e `claim` deve referenciar ao menos um `unit_id`
- Normalização de nomes: case folding, lematização/opcionais por idioma, remoção de sufixos ruídos
- Chaves de bloqueio por tipo de entidade (ex.: `Company:{name_norm,country}`; `Person:{name_norm,org?}`)
- Limites de orçamento: número máximo de unidades por lote e tokens por chamada
- Sensibilidade: se `TextUnit.sensitivity` for Confidencial, aplicar políticas de redação nos artefatos de IE exportados

## Saídas e Artefatos
- Artefato de IE por lote e por unidade: JSON versionado `v1`
- Estrutura normalizada:
  - `entities[]`: `{id?, type, name, aliases[], mentions:[{unit_id, span?}]}`
  - `relations[]`: `{id?, type, source{ref/id?}, target{ref/id?}, evidence:[unit_id], confidence}`
  - `claims[]`: `{id?, text, polarity, confidence, evidence:[unit_id]}`
- Métricas/traços: latência, custo, tokens in/out, modelo

## Parâmetros e Ajustes
- `model`, `temperature`, `max_output_tokens`, `batch_size`
- `min_confidence_entity`, `min_confidence_relation`, `min_confidence_claim`
- Estratégias de reforço (gleanings) para unidades densas: `enable_gleanings`, `gleanings_topk`

## Métricas e Critérios de Aceite
- Precisão e recall amostrais por tipo (entidade/relação/claim)
- Cobertura: % de `TextUnit` com ao menos uma extração
- Groundedness: 100% das saídas com `evidence.unit_id` válido
- Performance: custo/latência por 1k unidades dentro de orçamento

## Rastreabilidade
- Roadmap: 5) Modos de Consulta e Roteamento (base para Local/Global); 7) IE e Pós-Processo
- Roadmap técnico: 2.1 Ingestão (IE.Extract), 3) Modelos de Dados (grafo), 18.3 Contratos I/O (IEExtractorNode)

## Riscos e Considerações
- Ambiguidade de nomes (pessoas/empresas homônimas) → mitigação via chaves de bloqueio e contexto
- Ruído em PDFs digitalizados (OCR) → reduzir confiança mínima ou reprocessar com gleanings
- Idioma misto em um mesmo documento → perfis de prompt por idioma
- Custo variável por densidade informacional → fan-out controlado e cache de resultados idempotentes

## Exemplos
Entrada (`TextUnit` simplificada):
```json
{
  "unit_id": "u-000123",
  "doc_id": "d-42",
  "lang": "pt",
  "text": "A Contoso S.A. firmou parceria com a Fabrikam em 2024 para desenvolver IA responsável."
}
```

Saída esperada (IE v1):
```json
{
  "entities": [
    {"type": "Organization", "name": "Contoso S.A.", "aliases": ["Contoso"],
     "mentions": [{"unit_id": "u-000123"}]},
    {"type": "Organization", "name": "Fabrikam", "mentions": [{"unit_id": "u-000123"}]}
  ],
  "relations": [
    {"type": "Partnership", "source": {"name": "Contoso S.A."}, "target": {"name": "Fabrikam"},
     "evidence": ["u-000123"], "confidence": 0.91}
  ],
  "claims": [
    {"text": "Parceria estabelecida em 2024 para desenvolver IA responsável",
     "polarity": "positive", "confidence": 0.86, "evidence": ["u-000123"]}
  ]
}
```

## Variantes e Extensões
- Extração multimodal: associar `time_range` (áudio/vídeo) e `bbox` (imagem) nas evidências
- Gleanings seletivo: segunda passada apenas em unidades com alta densidade/entropia
- Tipos adicionais por domínio (regulatório, pessoas/processos) com normalização específica
