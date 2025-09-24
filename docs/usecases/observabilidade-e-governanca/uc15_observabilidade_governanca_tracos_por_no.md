# Traços por nó (latência, custo, tokens, provider/model)

## Contexto e Problema
Sem observabilidade detalhada por etapa do pipeline (SKGraph), fica difícil diagnosticar gargalos, estimar custos e explicar respostas. Precisamos de traços por nó com métricas de latência, tokens e custo, correlacionados por `traceId/queryId/projectId`, e com identificação do `provider/model` usado em cada chamada.

## Objetivo
Emitir e persistir um "trace" completo por execução de consulta/ingestão, contendo passos, nós executados, parâmetros-chave, custos e latências, com sobrecarga mínima e compatível com auditoria e XAI. Expor via API (`GET /v1/queries/{id}/trace`) e reaproveitar para métricas agregadas e A/B.

## Atores e Escopo
- Usuário/Cliente de API: solicita consulta e pode inspecionar o trace
- Serviços internos: nós SKGraph (`VectorRag`, `LocalGraph`, `GlobalQFS`, `DRIFT`, `IEExtractorNode`, etc.), telemetria e persistência
- Dentro: coleta de métricas em tempo de execução, correlação, serialização do trace e exposição via API
- Fora: renderizações avançadas (Mermaid/Neo4j Bloom/GraphView) tratadas no usecase de XAI

## Pré-condições
- Pipelines definidos via SKGraph com `GraphContext` contendo `traceId`, `projectId`, orçamento (`max_tokens_ctx`, `max_steps`, `max_level`)
- Hooks de telemetria habilitados nos nós para capturar `{latency_ms, tokens_in, tokens_out, cost, provider, model}`
- Armazenamento para traces (SQL e/ou objeto) e índices mínimos para consulta por `queryId`/`projectId`

## Entradas
- Execuções de ingestão/consulta disparadas via API (`POST /v1/query`) com `project_id`, `mode`, `filters`, `params`
- Eventos de nós SKGraph contendo métricas e metadados do passo

## Fluxo (alto nível)
- Etapa 1: Iniciar `GraphExecutionContext` com `traceId`, orçamento e políticas de amostragem de trace
- Etapa 2: Cada nó executa e emite evento de telemetria com métricas e I/O resumido (tamanhos, contagens, referências)
- Etapa 3: Agregar eventos ordenados, calcular custos totais e derivar indicadores (P50/P95 por nó/intent)
- Etapa 4: Persistir `trace` serializado (JSON) e índices para consulta rápida
- Etapa 5: Expor via `GET /v1/queries/{id}/trace` e publicar métricas agregadas
- Nós SKGraph (exemplos): `FunctionGraphNode`/`VectorRag`/`LocalGraph`/`GlobalQFS`/`DRIFT` com `RetryPolicyGraphNode` e `ConditionalGraphNode`

## Regras/Políticas e Guardrails
- Amostragem configurável (100% em DEV, 5–20% em PROD) por `tenant/project/intent`
- Redação automática de PII e dados sensíveis no trace quando necessário (integração com políticas do projeto)
- Orçamento respeitado: nós devem truncar contexto para manter `max_tokens_ctx`; registrar quando houver truncamento
- Idempotência e determinismo: repetir passos não deve duplicar métricas (agregação por `{projectId,nodeName,inputHash}`)

## Saídas e Artefatos
- `Trace` JSON com:
  - `queryId`, `traceId`, `projectId`, `userId`
  - `steps[]` com `{order, node, params, metrics{latency_ms,tokens_in,tokens_out,cost,provider,model}, outputs{sizes,citationsRefs}}`
  - `totals{latency_ms,cost,tokens_in,tokens_out}` e `budget{max_tokens_ctx,max_steps,max_level}`
- Persistência: tabela `QueryTrace` (SQL) + blob JSON em storage de objetos (`artifacts/traces/{queryId}.json`)
- Métricas agregadas para `GET /v1/metrics`

## Parâmetros e Ajustes
- `sampling_rate`, `include_io_samples` (quantidade de exemplos de I/O por nó), `max_steps`, `max_level`
- `redact_policy`: `{enabled, fields, sensitivity_levels}`
- `emit_provider_details`: incluir/ocultar `provider/model` conforme política

## Métricas e Critérios de Aceite
- Overhead de telemetria ≤ 5% na latência P95
- Cobertura: ≥ 99% das execuções com trace quando `sampling_rate=100%`
- Exatidão: soma de `tokens_in/out` e `cost` por passos ≈ totais (desvio ≤ 1%)
- API `GET /v1/queries/{id}/trace` retorna JSON válido e completo para consultas recentes

## Rastreabilidade
- Roadmap: seções 10 (Observabilidade), 5 (Modos de Consulta)
- Roadmap técnico: seções 7 (Observabilidade), 18 (Contratos de Nós), 6 (Endpoints)

## Riscos e Considerações
- Vazamento de dados sensíveis em amostras de I/O: aplicar redatores e limitar campos
- Volume/retention do trace: políticas de TTL e compactação
- Divergência de custos vs. faturamento do provedor: reconciliar via hooks do SDK do LLM

## Exemplos
Exemplo (trecho) de `trace` por consulta:
```json
{
  "queryId": "q-123",
  "projectId": "p-001",
  "mode": "global_graph",
  "totals": {"latency_ms": 2100, "tokens_in": 1600, "tokens_out": 320, "cost": 0.045},
  "steps": [
    {
      "order": 1,
      "node": "GlobalQFS",
      "metrics": {"latency_ms": 620, "tokens_in": 600, "tokens_out": 120, "cost": 0.015, "provider": "openai", "model": "gpt-4o-mini"},
      "outputs": {"sizes": {"selected_communities": 7}}
    },
    {
      "order": 2,
      "node": "MapReduceSummarize",
      "metrics": {"latency_ms": 980, "tokens_in": 800, "tokens_out": 160, "cost": 0.024, "provider": "openai", "model": "gpt-4o-mini"},
      "outputs": {"sizes": {"sections": 6}}
    }
  ]
}
```

## Variantes e Extensões
- Integração XAI: serializar subgrafo/trace ampliado (nós, arestas, passos) e renderizações (Mermaid/Neo4j Bloom/GraphView)
- A/B por intent/mode e exportação de métricas para sistemas de observabilidade (Prometheus/Grafana)
