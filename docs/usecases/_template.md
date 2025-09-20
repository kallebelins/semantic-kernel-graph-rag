# Padrão de saída Usecase: <Título do Usecase>

## Contexto e Problema
Descreva a dor/necessidade, motivação e quando este usecase é acionado.

## Objetivo
Resultado esperado e valor de negócio. O que muda quando bem-sucedido?

## Atores e Escopo
- Usuários/Serviços envolvidos
- Limites (o que está dentro/fora)

## Pré-condições
- Projetos, fontes e configurações necessárias
- Políticas e permissões mínimas

## Entradas
- Dados de entrada (documentos, consultas, filtros, parâmetros)
- Metadados relevantes (tags, setor, tema, idioma, sensibilidade)

## Fluxo (alto nível)
Liste as etapas principais. Quando aplicável, cite nós SKGraph esperados.
- Etapa 1: <descrição>
- Etapa 2: <descrição>
- Nós SKGraph (exemplos): `Normalizer`, `Chunker`, `Embedder`, `Bm25Indexer`, `IEExtractorNode`, `GraphMergeNode`, `CommunityDetect`, `ReportSummarize`, `VectorRag`, `LocalGraph`, `GlobalQFS`, `DRIFT`

## Regras/Políticas e Guardrails
- Regras de segurança/privacidade (RBAC/ABAC, PII, sensibilidade)
- Restrições funcionais e de qualidade

## Saídas e Artefatos
- Objetos produzidos (respostas, citações, relatórios, índices, métricas)
- Estruturas/formatos e onde são persistidos

## Parâmetros e Ajustes
Parâmetros relevantes e como impactam custo/qualidade:
- `k`, `rerank`, `max_tokens_ctx`, `max_communities`, `beam_width`, `depth`, etc.

## Métricas e Critérios de Aceite
- Métricas (latência, custo, relevância, cobertura, groundedness, diversidade)
- Critérios objetivos para considerar este usecase “pronto”

## Rastreabilidade
- Roadmap: consulte seções relacionadas em `docs/roadmap.md`
- Roadmap técnico: consulte seções relacionadas em `docs/roadmap_tecnico.md`

## Riscos e Considerações
- Riscos técnicos e operacionais
- Estratégias de mitigação

## Exemplos
- Exemplos de entrada/saída (quando aplicável)
- Amostras de configurações

## Variantes e Extensões
- Variações do fluxo (ex.: multimodal, cross-modal)
- Extensões futuras e integrações
