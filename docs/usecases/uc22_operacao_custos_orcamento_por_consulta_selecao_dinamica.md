# Padrão de saída Usecase: Orçamento por Consulta e Seleção Dinâmica de Contexto

## Contexto e Problema
Custos e latência variam com o tamanho de contexto, profundidade de busca e quantidade de passos no DAG. É preciso impor orçamento por consulta e abrir contexto de forma incremental apenas quando o ganho esperado justifica, evitando desperdício em perguntas simples e mantendo qualidade em perguntas complexas.

## Objetivo
Controlar `tokens`, `passos` e `níveis` por consulta, aplicando seleção dinâmica de comunidades/trechos e adaptando estratégias (Vector, Local, Global, DRIFT) para maximizar utilidade sob um teto de custo/tempo.

## Atores e Escopo
- Usuários: Finais e administradores de projeto.
- Serviços: `Router/DetectIntent`, `BudgetManager`, `GlobalQFS`, `LocalGraph`, `VectorRag`, `ReRanker`.
- Fora do escopo: Políticas de billing por tenant (tratadas na camada de operação/finops).

## Pré-condições
- Projeto com índices e (opcional) grafo/comunidades prontos.
- Perfis de prompt configurados e limites padrão por projeto/tenant.
- Parâmetros de orçamento habilitados no gateway (`max_tokens_ctx`, `max_steps`, `max_level`).

## Entradas
- `project_id`, `question`, `mode=auto|vector|local_graph|global_graph/qfs|drift`.
- Filtros: `tags[]`, `setor`, `tema`, `lang`, `sensitivity<=threshold`.
- Parâmetros: `k`, `rerank`, `max_tokens_ctx`, `max_steps`, `max_communities`, `beam_width`, `depth`, `max_level`.

## Fluxo (alto nível)
1) `DetectIntent` classifica a pergunta e define estratégia inicial e limites.
2) `BudgetManager` inicializa orçamento e instrumenta nós do DAG (contagem de tokens/steps/latência).
3) Execução incremental:
   - Vector: busca `(k1 BM25 ∪ k2 ANN)` mínima → `ReRanker` → resposta curta se confiança suficiente.
   - GlobalQFS: selecionar `n` comunidades, sintetizar e verificar cobertura; abrir mais `n` se orçamento permitir e ganho esperado justificar.
   - LocalGraph: expansão k-hop controlada por `beam_width`/`depth` e custo por hop.
   - DRIFT: global → subperguntas locais; interromper quando cobertura atingir limiar.
4) `BudgetManager` interrompe abertura quando atingir limites ou convergência.
5) `PIIFilter` aplica políticas e retorna resposta com citações e `trace`.

- Nós SKGraph (exemplos): `SwitchGraphNode`, `ConditionalGraphNode`, `ResultAggregator`, `RetryPolicyGraphNode`, `GlobalQFS`, `LocalGraph`, `VectorRag`, `FunctionGraphNode(BudgetManager)`.

## Regras/Políticas e Guardrails
- Respeitar `max_tokens_ctx`, `max_steps`, `max_level`; abortar com mensagem amigável se exceder.
- Seleção dinâmica baseada em `gain_expected` vs custo estimado (heurísticas por tipo de pergunta).
- RBAC/ABAC e sensibilidade antes de montar prompts; redigir PII.

## Saídas e Artefatos
- `answer`, `citations[]`, `trace` (subgrafo/etapas), `metrics {latency_ms, cost.tokens_in/out, steps}`.
- Logs por nó e decisão de abertura incremental.

## Parâmetros e Ajustes
- `max_tokens_ctx` (ex.: 6k–12k), `max_steps` (ex.: 8–20), `max_communities` (ex.: 4–16).
- `beam_width`, `depth` (LocalGraph); `rerank=true|false`; `k` inicial/expansão.
- Limiar de cobertura/groundedness para parada.

## Métricas e Critérios de Aceite
- Métricas: custo total de tokens, latência, passos executados, taxa de truncamento, cobertura.
- Aceite: consulta não excede limites; quando há truncamento, mensagem e citações mínimas presentes; traço completo disponível.

## Rastreabilidade
- Roadmap: seções 5 e 11 em `docs/roadmap.md` (seleção dinâmica e orçamento).
- Roadmap técnico: seções 2.2, 5, 7 e 9 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Subalocação pode reduzir qualidade; superalocação aumenta custo. Ajustar defaults por classe de pergunta.
- Estimativas de ganho imprecisas → coletar telemetria e ajustar heurísticas.

## Exemplos
- Consulta com orçamento:
```
POST /v1/query
{
  "project_id": "proj-123",
  "question": "Panorama de riscos de LGPD no último trimestre",
  "mode": "auto",
  "filters": { "lang": "pt" },
  "params": { "max_tokens_ctx": 6000, "max_communities": 6, "beam_width": 4, "depth": 2, "rerank": true }
}
```

## Variantes e Extensões
- A/B por modo/intent para refinar limites; adaptação por perfil do usuário.
- Orçamento dinâmico por SLA (executivo vs analista).
