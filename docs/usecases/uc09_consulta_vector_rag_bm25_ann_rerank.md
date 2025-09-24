# Vector RAG (BM25 ∪ ANN com re-rank)

## Contexto e Problema
Usuários fazem perguntas objetivas e específicas, com termos de vocabulário presentes nos conteúdos. É desejável baixa latência e custo, mantendo precisão e citações. A busca puramente lexical (BM25) pode falhar com sinônimos, enquanto a puramente vetorial (ANN) pode retornar trechos pouco ancorados lexicalmente ou irrelevantes em presença de termos raros. Precisamos combinar BM25 ∪ ANN e aplicar reclassificação (cross-encoder) para priorizar evidências mais relevantes.

## Objetivo
Entregar respostas curtas, precisas e citáveis a partir de trechos recuperados por BM25 e ANN, re-ranqueados por um cross-encoder, respeitando filtros de taxonomia e políticas de segurança, com baixa latência e custo.

## Atores e Escopo
- Usuários finais (via `POST /v1/query`)
- Serviços: Gateway de API, Núcleo (DAG/SKGraph), `IVectorIndex`, `ILexicalIndex`, `ReRanker`, `Answering/Summarizer`, `Guardrails`
- Dentro do escopo: recuperação, re-rank, síntese com citações, filtros/taxonomias, observabilidade
- Fora do escopo: expansão por grafo, comunidades, DRIFT (tratados em outros usecases)

## Pré-condições
- Projeto existente com índices atualizados: embeddings por `TextUnit` e índice BM25 populado
- Config: `PromptProfiles.answering`, `ReRanker` configurado, `rag_modes_enabled` inclui `vector`
- Políticas e permissões: RBAC/ABAC por `tenant/project/tag/setor/tema`; classificação de sensibilidade habilitada

## Entradas
- Requisição: `{ project_id, question, mode=vector, filters {tags[], setor, tema, lang}, params {k, rerank, max_tokens_ctx} }`
- Metadados relevantes por `TextUnit`: `{doc_id, unit_id, page, section, heading, tags, setor, tema, lang, sensitivity}`

## Fluxo (alto nível)
- Etapa 1: Roteador detecta intenção → segue para `VectorRag`
- Etapa 2: Recuperação híbrida
  - BM25: top `k_bm25`
  - Vetorial (ANN): top `k_vec`
  - União e deduplicação por `unit_id` (opcional balanceamento 50/50)
- Etapa 3: Re-rank (cross-encoder) nos candidatos para ordenar por relevância focada na pergunta
- Etapa 4: Seleção de contexto respeitando `params.max_tokens_ctx` (janelas, truncamento por prioridade)
- Etapa 5: Síntese curta (≤8 linhas) com citações `{doc_id, unit_id, page/section/heading}` e aplicação de `Guardrails`/políticas
- Etapa 6: Emissão de traços/metrics e resposta; cache opcional por `(project_id, question, filters, params)`
- Nós SKGraph (exemplos): `DetectIntent` → `VectorRag` (fan-in BM25/ANN) → `ReRanker` → `AnswerSynthesizer` → `Guardrails`

## Regras/Políticas e Guardrails
- Segurança: RBAC/ABAC aplicado a filtros; excluir `TextUnit` acima do nível de sensibilidade permitido
- Privacidade: redação/mascaramento de PII antes de compor a resposta
- Citações obrigatórias: incluir sempre `{doc_id, unit_id}` e, quando disponível, `page/section/heading`
- Limites: `max_tokens_ctx` (orçamento), `k` máximo e timeouts por nó

## Saídas e Artefatos
- `answer`: texto curto fundamentado
- `citations[]`: `{doc_id, unit_id, score, page?, section?, heading?, lang}`
- `trace`: nós, latência por etapa, tokens in/out, provider/model; amostras de entradas/saídas
- `metrics`: `latency_ms`, `cost.tokens_in/out`, `retrieval {bm25_hits, ann_hits, rerank_kept}`
- Cache: entrada/saída do re-rank e resposta final (respeitando políticas)

## Parâmetros e Ajustes
- `k`: total alvo de candidatos antes do re-rank (ex.: 40)
- `rerank`: habilita/seleciona cross-encoder (ex.: `ms-marco-MiniLM-L-6-v2`); `rerank.top_n` (ex.: 12)
- `max_tokens_ctx`: orçamento para contexto (ex.: 2k)
- Recomendações:
  - `k_bm25 = ceil(0.5 * k)`, `k_vec = ceil(0.5 * k)`; ajustar por domínio
  - Penalização de duplicatas por `doc_id` para diversidade
  - Filtros `{tags,setor,tema,lang}` aplicados nativamente nos índices para reduzir pós-processo

## Métricas e Critérios de Aceite
- Latência P50 ≤ 1.5s (sem re-rank) ou ≤ 2.5s (com re-rank), P95 ≤ 4s
- Groundedness ≥ 0.9 (respostas citam evidências corretas)
- Relevância (nDCG@10) ≥ baseline BM25 e ≥ baseline ANN isolados
- Custo por resposta dentro do orçamento médio do projeto

## Rastreabilidade
- Roadmap: seções 5 (Modos de Consulta), 6 (Fragmentação/Indexação), 10 (Observabilidade), 11 (Operação)
- Roadmap técnico: seções 5 (Modos), 6 (Endpoints `/v1/query`), 7 (Observabilidade), 10 (Serviços/Stores)

## Riscos e Considerações
- Drift de embeddings e termos raros: mitigar com BM25 ∪ ANN e atualização periódica
- Viés lexical excessivo: balancear `k_bm25` vs `k_vec` e aplicar re-rank robusto
- Conteúdo confidencial em contexto: aplicar filtros de sensibilidade e ABAC antes do re-rank/síntese
- Custos do re-rank: limitar `rerank.top_n`, cachear features e respostas frequentes

## Exemplos
- Requisição:
```json
{
  "project_id": "proj-123",
  "question": "Quais são os pré-requisitos do processo X?",
  "mode": "vector",
  "filters": {"setor": "Operações", "lang": "pt"},
  "params": {"k": 40, "rerank": true, "max_tokens_ctx": 2000}
}
```
- Resposta (resumo):
```json
{
  "answer": "Os pré-requisitos incluem A, B e C...",
  "citations": [
    {"doc_id":"docA","unit_id":"u123","page":3,"section":"2.1"},
    {"doc_id":"docB","unit_id":"u987","heading":"Requisitos"}
  ],
  "trace_id":"q-abc",
  "metrics": {"latency_ms": 1800, "retrieval": {"bm25_hits": 40, "ann_hits": 40, "rerank_kept": 12}}
}
```

## Variantes e Extensões
- Re-rank multicritério (relevância + groundedness + diversidade)
- Contexto multimodal (áudio com `time_range`, imagem com `bbox`) mantendo citações padronizadas
- Cache multicamadas: candidatos, escores de re-rank e respostas finais
- Integração com A/B para comparar configurações de `k`, `rerank` e prompts
