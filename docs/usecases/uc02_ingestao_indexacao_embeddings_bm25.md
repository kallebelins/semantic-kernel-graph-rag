# Usecase: Embeddings + BM25 (filtros por tags/setor/tema/lang)

## Contexto e Problema
Após fragmentação, é necessário indexar para recuperação híbrida (lexical + vetorial).

## Objetivo
Gerar embeddings e indexar postings BM25 com metadados facetados para filtros rápidos.

## Atores e Escopo
- Serviços: `EmbeddingService`, `BM25Service`
- Escopo: Indexação vetorial e lexical; sem resposta

## Pré-condições
- `TextUnit` persistidos
- Config de stores (Vector, BM25)

## Entradas
- `TextUnit[]` de um projeto
- Filtros padrão do projeto

## Fluxo (alto nível)
- Embedder.Run → `embeddings` com payloads
- Bm25Indexer.Run → postings facetados
- Nós SKGraph: `Embedder`, `Bm25Indexer`

## Regras/Políticas e Guardrails
- Respeitar limites de taxa/custo por tenant
- Excluir unidades confidenciais quando não permitido

## Saídas e Artefatos
- Índice vetorial (coleção por projeto)
- Índice BM25 com campos facetados

## Parâmetros e Ajustes
- `embedding_model`, `dim`, `batch_size`
- BM25: `k1`, `b`, campos indexados

## Métricas e Critérios de Aceite
- Latência e custo por 1k unidades
- Cobertura (percentual de unidades indexadas)

## Rastreabilidade
- Roadmap: 6) Estratégias de Fragmentação e Indexação
- Roadmap técnico: 2.1 Ingestão; 10) Serviços, Stores e Dependências

## Riscos e Considerações
- Drift de vocabulário por setor/tema
- Dimensões de embedding incompatíveis

## Exemplos
- Indexação com payload `{doc_id, unit_id, tags, setor, tema, lang}`

## Variantes e Extensões
- Indexação de `CommunityReport` em coleções próprias
