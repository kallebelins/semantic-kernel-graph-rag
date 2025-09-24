# Parâmetros e filtros `{k, rerank, max_tokens_ctx, max_communities, beam_width, depth}`

## Contexto e Problema
Os modos de consulta precisam operar sob limites de custo/latência e respeitar políticas e recortes de contexto (tags, setor, tema, idioma). Sem padronização de parâmetros, o comportamento fica imprevisível e caro.

## Objetivo
Definir parâmetros e filtros suportados, sua semântica e impacto em custo/qualidade; padronizar aplicação nos nós do DAG e na API.

## Atores e Escopo
- Usuário (via `POST /v1/query`)
- Serviços: Gateway/API, DAG (Vector/Local/Global/DRIFT), Observabilidade
- Dentro: definição, defaults recomendados, validação e aplicação coerente
- Fora: detalhes de implementação de índices/grafo (cobertos em outros usecases)

## Pré-condições
- Índices e grafos disponíveis; políticas de segurança carregadas no contexto
- Configuração por projeto: limites máximos por parâmetro e modos habilitados

## Entradas
- `filters`: `{tags[], setor, tema, lang}`
- `params`: `{k, rerank, max_tokens_ctx, max_communities, beam_width, depth}`

## Fluxo (alto nível)
- Etapa 1: Validar filtros e aplicar restrições por ABAC/RBAC
- Etapa 2: Normalizar parâmetros a limites por projeto e por modo
- Etapa 3: Propagar parâmetros no `GraphContext.Budget` para todos os nós
- Etapa 4: Registrar métricas de consumo e efetividade por parâmetro

## Regras/Políticas e Guardrails
- `max_tokens_ctx`: orçamento total de contexto; nós devem respeitar cortes/truncamentos
- `k`: candidatos de recuperação antes de re-rank/síntese; limitar por modo
- `rerank`: habilitar cross-encoder e `rerank.top_n`; registrar custo
- `max_communities`: limite de comunidades abertas em QFS/DRIFT
- `beam_width`/`depth`: controle de expansão em Local/DRIFT
- Filtros aplicados nativamente aos índices/grafo para minimizar pós-processo

## Saídas e Artefatos
- Resposta dos modos respeitando filtros e orçamento
- `metrics.params_effect`: impacto de cada parâmetro em latência/custo/qualidade

## Parâmetros e Ajustes (defaults recomendados)
- Vector: `k=40`, `rerank=true`, `rerank.top_n=12`, `max_tokens_ctx=2000`
- Local: `depth=2..3`, `beam_width=5..8`, `k_evidence=20..40`, `max_tokens_ctx=2000`
- Global (QFS): `max_communities=5..10`, `beam_width=3..5`, `max_tokens_ctx=1500`
- DRIFT: `max_communities=3..6`, `k_sub=20..40`, `rerank.top_n=12..16`, `max_tokens_ctx=2200`

## Métricas e Critérios de Aceite
- Parâmetros são respeitados por todos os nós (sem estouro de orçamento)
- Observabilidade reporta consumo por parâmetro e correlação com qualidade
- Experimentos A/B mostram curvas custo×qualidade estáveis

## Rastreabilidade
- Roadmap: seções 5 (Modos), 11 (Operação), 10 (Observabilidade)
- Roadmap técnico: seções 5 (Parâmetros), 7 (Métricas), 18 (Budget no contexto)

## Riscos e Considerações
- Parâmetros mal calibrados podem reduzir groundedness ou aumentar custo
- Defaults inadequados por domínio/idioma: permitir override por projeto

## Exemplos
- Exemplo de `POST /v1/query` com filtros e params já mostrado em usecases de modo

## Variantes e Extensões
- Política adaptativa: ajustar parâmetros em tempo de execução por ganho esperado
