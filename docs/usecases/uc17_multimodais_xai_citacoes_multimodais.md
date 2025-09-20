# Usecase: Citações multimodais: `time_range`, `bbox`, `page/section/heading`

## Contexto e Problema
Respostas confiáveis exigem apontar exatamente onde a evidência está. Em ambientes corporativos e regulados, isto inclui trechos de documentos (página/seção/heading), regiões de imagem (bbox OCR) e segmentos de áudio/vídeo (time range e diarização do falante). Sem um modelo único de citação multimodal, torna-se difícil auditar, aplicar políticas (PII/sensibilidade) e reutilizar evidências em relatórios e traces.

## Objetivo
Padronizar, propagar e exigir citações multimodais de ponta a ponta (ingestão → indexação → consulta → resposta), garantindo que toda resposta decisória inclua referências precisas ao conteúdo-fonte com metadados normalizados por mídia.

## Atores e Escopo
- Usuário final via `POST /v1/query`
- Serviços: `Normalizer`, `Chunker`, `OCR/ASR`, `Embedder`, `BM25`, `VectorRag`/`LocalGraph`/`GlobalQFS`, `ReRanker`, `Guardrails`, `PIIFilter`
- Dentro: representação de citações, coleta e propagação em todos os nós de pipeline, exibição no output e persistência
- Fora: UI de renderização detalhada (tratada no console), mas o formato é preparado para ela

## Pré-condições
- Pipelines de ingestão habilitados para extrair metadados por mídia:
  - Documentos: `page`, `section`, `heading`
  - Imagens/PDF escaneado: OCR com `bbox` por trecho
  - Áudio/Vídeo: ASR com `time_range` (`hh:mm:ss-hh:mm:ss`) e `speaker_label`
- Armazenamento de metadados em `TextUnit` e payloads de índices (vetorial e BM25)
- Políticas e perfis de prompt com exigência de citações para respostas decisórias

## Entradas
- Consulta: `{ project_id, question, mode, filters {tags,setor,tema,lang}, params {...} }`
- Artefatos de ingestão: `TextUnit{ unit_id, doc_id, page, section, heading, lang, sensitivity }`, OCR (`bbox`), ASR (`time_range`, `speaker_label`)

## Fluxo (alto nível)
- Etapa 1: Ingestão normaliza e fragmenta preservando metadados de mídia
- Etapa 2: Indexação vetorial e BM25 incluindo payload de citação multimodal
- Etapa 3: Recuperação (Vector/Local/Global) retorna candidatos com payloads completos de citação
- Etapa 4: Re-rank e síntese montam resposta curta (≤8 linhas) com citações obrigatórias
- Etapa 5: Guardrails/Políticas aplicam RBAC/ABAC, sensibilidade e redação/mascaramento de PII nas citações
- Nós SKGraph: `Normalizer` → `Chunker` → (`OCR`/`ASR` opcionais) → `Embedder` → `Bm25Indexer` → `{VectorRag|LocalGraph|GlobalQFS}` → `ReRanker` → `PIIFilter` → `Guardrails`

## Regras/Políticas e Guardrails
- Citações obrigatórias para respostas decisórias, contendo, conforme a mídia:
  - Documento: `{doc_id, unit_id, page, section, heading}`
  - Imagem: `{doc_id, unit_id, bbox}`
  - Áudio/Vídeo: `{doc_id, unit_id, time_range, speaker_label}`
- Redação/mascaramento quando `sensitivity ∈ {Interno, Confidencial}` e o usuário não possuir escopo
- Quantidade máxima de citações por resposta e por mídia para controlar custo e legibilidade

## Saídas e Artefatos
- `answer` com bloco `citations[]` multimodal
- `trace` com referências a `TextUnit`/`community_id` e métricas por nó
- Persistência:
  - Citations serializadas junto do resultado da `Query` (SQL) e em objetos (JSON) para reuso/BI
  - Payloads mantidos em índices vetoriais/BM25 para reexecuções determinísticas

Estrutura sugerida de citação (union por mídia):
```json
{
  "citations": [
    { "type": "document", "doc_id": "D123", "unit_id": "U987", "page": 12, "section": "2.3", "heading": "Requisitos" },
    { "type": "image", "doc_id": "D124", "unit_id": "U990", "bbox": [120, 340, 420, 560] },
    { "type": "av", "doc_id": "D200", "unit_id": "U450", "time_range": "00:12:40-00:13:05", "speaker_label": "spk_1" }
  ]
}
```

## Parâmetros e Ajustes
- `max_citations_total` (ex.: 6–10), `max_citations_per_media` (ex.: 3)
- `include_thumbnails` para imagens e `waveform_snippets` para áudio/vídeo (opcional)
- `citation_style`: curto|detalhado (inclui heading/trecho textual curto)

## Métricas e Critérios de Aceite
- Groundedness ≥ alvo (todas as afirmações chave referenciadas)
- Cobertura de citação ≥ 80% para sentenças factuais na resposta
- Latência P95 ≤ 2.5s adicional vs. versão sem citações multimodais
- Conformidade: bloqueio de exibição quando política exigir e escopo ausente

## Rastreabilidade
- Roadmap: seções 4 (Citações e Metadados Multimodais), 9 (Segurança/Privacidade), 10 (Observabilidade)
- Roadmap técnico: seções 4 (Citações e Metadados), 6 (Endpoints de API – Consulta), 7 (Observabilidade), 18 (Contratos I/O)

## Riscos e Considerações
- Erros de OCR/ASR prejudicam precisão de `bbox`/`time_range` → Mitigar com qualidade mínima e validação amostral
- Explosão de citações em respostas longas → Limitar e priorizar por relevância/novidade
- Privacidade de falas (speaker) → Anonimizar `speaker_label` por política

## Exemplos
- Pergunta: "Onde está a política de backup descrita e quem a anunciou?"
- Resposta: curta com 2 citações de documento (página/heading) e 1 de vídeo (time range do anúncio)

## Variantes e Extensões
- Renderização enriquecida na UI: highlights por `bbox` e scrubbing por `time_range`
- Agregação por entidade/claim: citações agrupadas por ponto da resposta
- Exportação para BI: fatos com referências multimodais para auditoria
