# Usecase: Detecção hierárquica (Leiden/Louvain) por nível

## Contexto e Problema
A plataforma consolida um grafo de conhecimento por projeto (entidades, relações e claims). Para habilitar visão global, navegação macro e sumarização bottom-up, é necessário particionar o grafo em comunidades hierárquicas (níveis 0..n). O desafio é detectar módulos de forma eficiente, estável e idempotente, suportando diferentes tamanhos de grafo e restrições de orçamento.

## Objetivo
- Gerar partições hierárquicas do grafo por nível (fino → macro) com qualidade medível (ex.: modularidade) e persistir nós `Community` e arestas `(:Entity)-[:IN_COMMUNITY {level}]->(:Community)`.
- Produzir métricas por nível e preparar insumos para relatórios de comunidade e QFS (usecases seguintes).

## Atores e Escopo
- Serviços: `CommunityService` (núcleo .NET), `GraphStore` (Neo4j), opcional microserviço Python (`leidenalg/igraph`).
- Escopo: detecção de comunidades e persistência dos artefatos; geração de relatórios fica no UC08.

## Pré-condições
- Subgrafo do projeto disponível (`GraphSnapshot`).
- Constraints no Neo4j criadas para `Community` (id único) conforme `docs/roadmap_tecnico.md`.
- Parâmetros de execução definidos (níveis, resolução, iterações, limites de orçamento).

## Entradas
- `project_id` e `snapshotRef` ou `edges` do subgrafo atual.
- Parâmetros: `levels (int)`, `resolution (float)`, `iterations (int)`, `min_community_size (int)`, `weighted (bool)`.
- Metadados/filtros (opcional): `tags`, `setor`, `tema`, `lang` para focar subgrafos.

## Fluxo (alto nível)
- Etapa 1: Obter `GraphSnapshot` do projeto via `GraphStore.GetSubgraphAsync(projectId)`.
- Etapa 2: Para cada `level` em `0..levels-1`, executar detecção com parâmetros (ex.: `resolution` ajustado por nível).
- Etapa 3: Persistir `Community {id, level, projectId}` e arestas `IN_COMMUNITY {level}`; salvar métricas (modularidade, cobertura).
- Etapa 4: (Opcional) Refinar partições (ex.: conductance/triangles) e filtrar comunidades abaixo de `min_community_size`.
- Etapa 5: Registrar telemetria por execução (latência, custo, modelo/provedor se usar LLM em etapas subsequentes — aqui normalmente não há LLM).
- Nós SKGraph (exemplos): `FunctionGraphNode(GetSnapshot)`, `CommunityDetect(level)`, `ResultAggregator(SaveLevel)`, `RetryPolicyGraphNode`.

## Regras/Políticas e Guardrails
- Idempotência: chavear resultados por `{projectId, level, snapshotHash, paramsHash}` para evitar retrabalho.
- Determinismo: mesma entrada → mesma partição (fixar seed no serviço Python quando aplicável).
- Orçamento: respeitar `max_steps` e tempo total; fallback automático para Louvain .NET se microserviço estiver indisponível.
- Segurança: respeitar escopos por projeto/tenant; não expor dados entre projetos.
- Observabilidade: traços por nível com métricas e eventuais erros serializados.

## Saídas e Artefatos
- `partitions`: lista de comunidades com seus membros por nível.
- `metrics`: por nível (ex.: modularidade, número de comunidades, cobertura de nós/arestas, tamanho mínimo/médio/máximo).
- Persistência no grafo: nós `(:Community {id, level, projectId, summary?})` e arestas `(:Entity)-[:IN_COMMUNITY {level}]->(:Community)`.
- Registro para próximos usecases: referências de `communityIds` para sumarização (UC08) e embeddings de relatórios.

## Parâmetros e Ajustes
- `levels`: número de níveis hierárquicos (típico 2–4).
- `resolution`: controla granularidade (maior → mais comunidades); pode variar por nível (ex.: `1.2, 0.9, 0.6`).
- `iterations`: iterações do algoritmo (10–50, conforme custo/estabilidade).
- `min_community_size`: poda comunidades muito pequenas.
- `weighted`: usar pesos de aresta (ex.: contagem de co-ocorrências ou força da relação).

## Métricas e Critérios de Aceite
- Métricas: `latency_ms`, `modularity`, `num_communities`, `coverage_nodes`, `coverage_edges`, `size_min/avg/max`.
- Critérios de aceite:
  - Executa para `levels ≥ 2` com modularidade acima de um limiar configurável.
  - Persistência correta de nós `Community` e arestas `IN_COMMUNITY` por nível.
  - Idempotência verificada em reexecução com mesmo snapshot/params.

## Rastreabilidade
- Roadmap: `docs/roadmap.md` seções 8 (Comunidades e Relatórios) e 6 (Estratégias/Indexação, como base de pesos).
- Roadmap técnico: `docs/roadmap_tecnico.md` seções 2.3 (Comunidades), 3.2 (Modelo de Grafo), 7 (Observabilidade) e 18 (Contratos de nós).

## Riscos e Considerações
- Qualidade de partição ruim em grafos muito esparsos/densos; mitigação: ajuste de `resolution` e pré-processo de arestas.
- Indisponibilidade do microserviço Leiden; mitigação: fallback Louvain .NET.
- Custo/tempo em grafos grandes; mitigação: amostragem/limitação por nível ou por componentes conexas.
- Drift estrutural entre rodadas; manter snapshot/params para auditoria e reprodutibilidade.

## Exemplos
Parâmetros de execução por nível (conceitual):
```json
{
  "levels": 3,
  "resolution": [1.2, 0.9, 0.6],
  "iterations": 20,
  "min_community_size": 6,
  "weighted": true
}
```

Chamada ao microserviço Leiden (conceitual):
```http
POST http://leiden-svc/communities
Content-Type: application/json

{
  "edges": [{"source": "E1", "target": "E2", "weight": 0.8}],
  "resolution": 1.0,
  "iterations": 10
}
```

Persistência (Cypher conceitual):
```cypher
MERGE (m:Community {id: $communityId, level: $level, projectId: $projectId})
WITH m
UNWIND $members AS entityId
MATCH (e:Entity {id: entityId, projectId: $projectId})
MERGE (e)-[:IN_COMMUNITY {level: $level}]->(m);
```

## Variantes e Extensões
- Pesos de aresta por tipo de relação (ex.: `MENTIONED_IN` vs `REL`) ou força textual.
- Refinamento local pós-detecção (conductance, triângulos).
- Execução incremental por diff (reprocesso seletivo por fonte/modificação de arestas).
- Integração com camada social/organizacional (UC25) como atributos para pesos/seed de comunidades.
