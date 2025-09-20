# Usecase: Global GraphRAG (QFS por comunidades)

## Contexto e Problema
Perguntas de panorama, visão geral, tendências ou “mapa do tema” exigem navegar conhecimento agregado. Percorrer documentos ou subgrafos locais é ineficiente e caro. Precisamos de uma busca global focada em consulta (QFS) baseada em comunidades hierárquicas e seus relatórios, com navegação bottom-up.

## Objetivo
Selecionar comunidades relevantes em múltiplos níveis, consumir `CommunityReport` embeddeds e executar um fluxo tipo map-reduce para produzir visão global coerente e citável, com baixo custo e boa cobertura.

## Atores e Escopo
- Usuário (via `POST /v1/query`, `mode=global_graph` ou `mode=qfs`)
- Serviços: `ICommunityService`, `IReportService`, `GlobalQFS`, `Summarizer`, `Guardrails`, `IVectorIndex` para embeddings de relatórios
- Dentro: seleção de comunidades, navegação hierárquica, sumarização multi-nível, citações por `community_id` e evidências representativas
- Fora: expansão local detalhada (coberta por Local GraphRAG) e híbrido (DRIFT)

## Pré-condições
- Comunidades detectadas por nível (Leiden/Louvain); `CommunityReport` gerado e indexado (embeddings)
- Config de QFS: limites por `max_communities`, `beam_width`, `max_tokens_ctx`
- `PromptProfiles.summarization` para estilo executivo e seções padronizadas

## Entradas
- `{ project_id, question, mode=global_graph, filters {tags,setor,tema,lang}, params {max_communities, beam_width, max_tokens_ctx} }`
- Índice vetorial contendo embeddings de `CommunityReport` com payload `{community_id, level, centroidEntities[], summary}`

## Fluxo (alto nível)
- Etapa 1: Buscar top-N `CommunityReport` por embeddings (consulta vetorial da pergunta)
- Etapa 2: Selecionar níveis e abrir hierarquia sob demanda (bottom-up) conforme orçamento
- Etapa 3: Map: extrair pontos-chave por comunidade selecionada; Reduce: consolidar por tema/seção
- Etapa 4: Síntese executiva em 6–10 linhas com citações por `community_id` e, quando útil, por entidades centrais
- Etapa 5: Guardrails/políticas e emissão de traço/métricas
- Nós SKGraph: `GlobalQFS` → `CommunitySelect` → `HierarchicalOpen` → `MapReduceSummarize` → `Guardrails`

## Regras/Políticas e Guardrails
- Respeitar filtros e sensibilidade, ocultando comunidades que reflitam conteúdo restrito
- Evitar redundância: penalizar comunidades com alto overlap de entidades centrais
- Seleção dinâmica: abrir mais níveis apenas se ganho esperado justificar custo

## Saídas e Artefatos
- `answer` panorâmica com citações `{community_id, level}` e links para entidades centrais
- `selected_communities[]` com escores, níveis e porções abertas
- `metrics`: `{latency_ms, tokens_in/out, selected_levels, opened_communities}`
- Cache de `CommunityReport` e de resultados QFS

## Parâmetros e Ajustes
- `max_communities` (ex.: 5–10), `beam_width` (ex.: 3–5), `max_tokens_ctx` (ex.: 1500)
- Estratégias de abertura: breadth-first por nível vs. seleção por centralidade/novidade

## Métricas e Critérios de Aceite
- Cobertura de temas (diversidade) superior a Vector/Local sob mesmo orçamento
- Latência P95 ≤ 3.5s; custo dentro de limites do projeto
- Aderência ao estilo executivo do `CommunityReport` e groundedness com referências

## Rastreabilidade
- Roadmap: seções 5 (Global GraphRAG), 8 (Comunidades/Relatórios)
- Roadmap técnico: seções 2.3 (Comunidades), 3.2 (Community/Report), 5 (Modos), 7 (Observabilidade)

## Riscos e Considerações
- Comunidades desbalanceadas/ruído: ajustar resolução/iterações do detector e refinamento
- Relatórios desatualizados: reprocessamento incremental após atualização do grafo
- Tendência a generalidades: reforçar prompts com exigência de citações e objetivos/riscos/lacunas

## Exemplos
- Pergunta: "Qual o panorama dos riscos de compliance no último trimestre?"
- Saída: visão por comunidades ligadas a LGPD/ISO, com 3 pontos-chave e 3 lacunas citadas por `community_id`

## Variantes e Extensões
- QFS hierárquico com níveis adaptativos conforme confiança
- Exportação dos resultados para playbooks de Insights (BI)
