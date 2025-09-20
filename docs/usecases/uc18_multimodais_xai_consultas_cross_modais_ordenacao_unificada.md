# Usecase: Consultas cross-modais e ordenação unificada

## Contexto e Problema
Usuários fazem perguntas que combinam evidências em texto, imagens e gravações. Executar três buscas separadas sem uma ordenação unificada causa respostas desalinhadas, com vieses por modalidade e baixa coerência. Precisamos de um fluxo cross-modal com roteamento explícito, recuperação paralela por modalidade e reclassificação unificada para compor uma única resposta fundamentada.

## Objetivo
Habilitar `Intent.CrossModal` no roteador e implementar um pipeline que:
1) recupere candidatos por modalidade, 2) normalize seus scores, 3) aplique re-rank cross-encoder multimodal, e 4) sintetize uma resposta única com citações heterogêneas.

## Atores e Escopo
- Usuário via `POST /v1/query` (`mode=auto` com roteador ou `intent=cross_modal`)
- Serviços: `IntentClassifier`, `VectorRetrievers (text/image/audio)`, `BM25`, `CrossModalReRanker`, `Summarizer`, `Guardrails`
- Dentro: detecção de intenção, recuperação paralela, fusão/ordenação, síntese e citações
- Fora: treinamento de modelos CLIP/ASR/OCR (assumidos como serviços externos)

## Pré-condições
- Índices por modalidade disponíveis:
  - Texto: embeddings + BM25 com payloads de citação (page/section/heading)
  - Imagem: embeddings (ex.: CLIP) com OCR opcional (`bbox`)
  - Áudio/Vídeo: transcrição (ASR) indexada e metadados (`time_range`, `speaker_label`)
- Parâmetros do roteador habilitando `Intent.CrossModal`
- Re-ranker multimodal (cross-encoder) configurado

## Entradas
- `{ project_id, question, mode=auto|cross_modal, filters {tags,setor,tema,lang}, params {k_text,k_image,k_av,weights,rerank} }`
- Coleções/índices por modalidade com payloads de citação multimodal

## Fluxo (alto nível)
- Etapa 1: `DetectIntent` identifica `CrossModal` (por vocabulário/estrutura da pergunta)
- Etapa 2: Recuperadores paralelos: texto (ANN ∪ BM25), imagem (ANN), áudio/vídeo (ANN em transcrições)
- Etapa 3: Normalização de scores (z-score/min-max) e fusão de candidatos em lista única
- Etapa 4: `CrossModalReRanker` (cross-encoder) reclassifica considerando contexto da pergunta e características por mídia
- Etapa 5: `Summarizer` produz resposta curta com citações multimodais; `Guardrails` aplica políticas
- Nós SKGraph: `DetectIntent` → `FanOut {RetrieveText, RetrieveImage, RetrieveAV}` → `Join(FuseAndNormalize)` → `CrossModalReRanker` → `Summarizer` → `Guardrails`

## Regras/Políticas e Guardrails
- Balanceamento por modalidade via `weights` e limite por `max_citations_per_media`
- Respeito a RBAC/ABAC e sensibilidade por mídia; anonimização de `speaker_label` quando exigido
- Deduplicação por unidade/semântica (ex.: transcrição vs. documento fonte similar)

## Saídas e Artefatos
- `answer` única com `citations[]` de diferentes mídias
- `candidates[]` pós-fusão com `score_before`/`score_after_rerank` para auditabilidade
- Métricas de contribuição por modalidade e latência por ramo paralelo

## Parâmetros e Ajustes
- `k_text`, `k_image`, `k_av` (ex.: 20/10/10) e `k_final` pós-rerank
- `weights {text,image,av}` (ex.: `{0.5,0.2,0.3}`) para fusão inicial
- `rerank=model_name` e `max_tokens_ctx` para síntese

## Métricas e Critérios de Aceite
- Melhoria de precisão@k em consultas com termos visuais/sonoros vs. somente texto
- Diversidade por modalidade ≥ alvo (ex.: ≥ 2 mídias presentes quando aplicável)
- Latência P95 ≤ 3.5s com fan-out paralelo e re-rank unificado

## Rastreabilidade
- Roadmap: seções 5 (Modos de Consulta), 12/16 (Cross-modais), 6 (Fragmentação/Indexação)
- Roadmap técnico: seções 5 (Modos), 12 (Prompts/Few-Shots – CrossModal), 18 (Contratos I/O)

## Riscos e Considerações
- Model mismatch entre embeddings (texto vs. imagem) → utilizar espaço comum (CLIP) e calibrar normalização
- Ruído de ASR/OCR → validar confiança mínima e filtrar segmentos curtos/ambíguos
- Custo do cross-encoder → cache de features e reuso por janela de tempo

## Exemplos
- Pergunta: "Qual documento descreve o diagrama desta figura e em qual reunião foi aprovado?"
- Resposta: cita página/heading do documento e time range + `speaker_label` do trecho de aprovação na reunião

## Variantes e Extensões
- Fallback adaptativo: se baixa cobertura em uma mídia, redistribuir orçamento para as demais
- Integração com Local GraphRAG após fusão para coletar evidências estruturadas relacionadas
