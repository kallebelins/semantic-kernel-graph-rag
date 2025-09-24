# DRIFT (global → local com reclassificação)

## Contexto e Problema
Perguntas longas/ambíguas ou que pedem visão geral com detalhes pontuais exigem combinação de global e local. Só QFS pode ficar genérico; só Local pode perder cobertura. Precisamos de um fluxo híbrido: filtra globalmente, gera subperguntas locais e reclassifica o conjunto final.

## Objetivo
Alcançar equilíbrio entre cobertura e precisão: usar QFS para reduzir o espaço de busca, executar subconsultas locais direcionadas e reclassificar evidências/partes para compor resposta única com alta groundedness.

## Atores e Escopo
- Usuário (via `POST /v1/query`, `mode=drift` ou `mode=auto` com roteador)
- Serviços: `GlobalQFS`, `QuestionDecomposer`, `LocalGraph`, `VectorRag` (fallback), `ReRanker`, `AnswerSynthesizer`, `Guardrails`
- Dentro: seleção global, decomposição de pergunta, consultas locais, fusão e re-rank final
- Fora: geração de relatórios de comunidade (pré-requisito do QFS)

## Pré-condições
- Comunidades/relatórios e índices prontos (como em QFS/Local)
- `PromptProfiles`: `summarization`, `answering`, `classification` (para decomposição e fusão)

## Entradas
- `{ project_id, question, mode=drift, filters {...}, params {max_communities, depth, beam_width, k, rerank, max_tokens_ctx} }`

## Fluxo (alto nível)
- Etapa 1: QFS (global) seleciona comunidades relevantes (limite por `max_communities`)
- Etapa 2: Decompor pergunta em subperguntas por comunidade/tema
- Etapa 3: Para cada subpergunta, escolher estratégia local (Local Graph preferencial; Vector como fallback) e executar com limites
- Etapa 4: Agregar evidências e respostas parciais; re-rank multicritério e deduplicação
- Etapa 5: Síntese final única com citações heterogêneas (community_id e unit_id) e guardrails
- Nós SKGraph: `DRIFT` → `CommunitySelect(QFS)` → `Decompose` → `FanOut{Local/Vector}` → `Join+ReRank` → `AnswerSynthesizer`

## Regras/Políticas e Guardrails
- Orçamento global: dividir `max_tokens_ctx` entre fases; abortar fan-out se orçamento apertado
- Segurança/privacidade: filtros aplicados em todas as fases; citações obrigatórias
- Decomposição conservadora: limitar número de subperguntas por comunidade

## Saídas e Artefatos
- `answer` única e coesa
- `citations[]`: mistura de `{community_id}` e `{doc_id, unit_id, ...}`
- `trace`: fases globais/locais, subperguntas e fusão; métricas por fase

## Parâmetros e Ajustes
- `max_communities` (2–6), `k` por subconsulta (20–40), `depth/beam_width` para Local, `rerank.top_n` (12–16)
- Estratégias de fusão: priorizar evidências com maior suporte e diversidade de origem

## Métricas e Critérios de Aceite
- Melhora de cobertura vs Local isolado e melhora de precisão vs QFS isolado
- Latência P95 ≤ 5–6s sob limites padrão
- Groundedness ≥ 0.9 e diversidade de fontes/comunidades

## Rastreabilidade
- Roadmap: seções 5 (DRIFT), 8 (Comunidades/Relatórios), 10 (Observabilidade)
- Roadmap técnico: seções 5 (Modos), 2.2/2.3 (DAG/Comunidades), 18 (FanOut/Join)

## Riscos e Considerações
- Explosão de subperguntas: limite de fan-out e heurísticas de confiança
- Custo elevado: seleção dinâmica e cache de resultados QFS e parciais locais
- Contradições entre evidências: prompts para reconciliar e apontar incertezas

## Exemplos
- Pergunta: "Dê um panorama das iniciativas de segurança e cite exemplos práticos recentes"
- Fluxo: QFS seleciona comunidades → 3 subperguntas locais → fusão e re-rank → resposta única

## Variantes e Extensões
- Inclusão de Vector RAG para subperguntas puramente textuais
- Aprendizado de políticas de decomposição com base em traces anteriores
