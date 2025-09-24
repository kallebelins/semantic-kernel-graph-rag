# Upload/ZIP/GDrive → Normalize/Chunk (heading-aware, semantic split, rolling window)

## Contexto e Problema
Projetos precisam ingerir documentos heterogêneos e produzir unidades de texto coerentes para indexação e IE.

## Objetivo
Normalizar conteúdos e fragmentar em `TextUnit` preservando estrutura (títulos, seções, páginas) e coesão semântica.

## Atores e Escopo
- Usuários: Admin/Operador de Ingestão
- Serviços: Gateway API, Workers de Ingestão
- Escopo: Upload e normalização + chunking; sem embeddings/IE

## Pré-condições
- Projeto criado com taxonomias e políticas
- Fonte registrada (upload/zip/gdrive)

## Entradas
- Arquivos (PDF, DOCX, TXT, Markdown, ZIP)
- Metadados: `tags`, `setor`, `tema`, `lang`

## Fluxo (alto nível)
- Detect → Normalize (texto + metadados estruturais)
- Chunk (heading-aware + semantic split + rolling window)
- Persistir `Document` e `TextUnit`
- Nós SKGraph: `Normalizer`, `Chunker`

## Regras/Políticas e Guardrails
- Respeitar limites de tamanho por unidade e mínimo de sentença
- Preservar tabelas/código como blocos indivisíveis quando configurado

## Saídas e Artefatos
- `Document` e `TextUnit` persistidos
- Traços: latência e contagem de tokens por etapa

## Parâmetros e Ajustes
- `max_chunk_tokens`, `overlap_tokens`, `min_sentence_size`
- `preserve_tables`, `preserve_code_blocks`

## Métricas e Critérios de Aceite
- Latência por 100 páginas
- Coesão dos chunks (baixa taxa de cortes duros)

## Rastreabilidade
- Roadmap: 4) Pipeline de Ingestão; 6) Estratégias de Fragmentação
- Roadmap técnico: 2.1 Ingestão; 6) Endpoints (upload)

## Riscos e Considerações
- PDFs com OCR ruim; tabelas difíceis
- Idiomas mistos por documento

## Exemplos
- Upload ZIP com PDFs → `TextUnit` com `heading/page/section`

## Variantes e Extensões
- Sincronização GDrive/HTTP
- Suporte multimídia (áudio/imagem) na normalização
