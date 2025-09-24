# Exportações para Data Lake/BI e Dashboards

## Contexto e Problema
Os resultados do RAG híbrido (entidades, relações, claims, comunidades, relatórios e insights) precisam alimentar data lakes e ferramentas de BI (PowerBI/Tableau/Elastic) para análise contínua, compliance e governança. É necessário padronizar formatos, versionar por projeto/idioma, controlar custos e respeitar políticas de segurança/privacidade.

## Objetivo
Disponibilizar exportações confiáveis, incrementais e auditáveis dos artefatos gerados (tabelas relacionais, JSONs, grafos e métricas) para camadas analíticas, habilitando dashboards, KPIs e exploração ad-hoc. As exportações devem ser reprodutíveis, com controle de sensibilidade e trilha de auditoria.

## Atores e Escopo
- Usuários: Eng. de Dados/BI, Analistas, Gestores, Compliance.
- Serviços: `ExportService` (jobs), `GraphStore`, `IVectorIndex`, `ILexicalIndex`, `IObjectStorage`, `IProjectRepository`.
- Fora do escopo: Modelagem de dashboards (detalhes de PowerBI/Tableau) e automações específicas de cada ferramenta; coberto apenas por exemplos e contratos.

## Pré-condições
- Projeto processado (ingestão → IE → grafo → comunidades → relatórios → insights opcionais).
- Políticas de RBAC/ABAC e classificação de sensibilidade ativas.
- Configuração de destinos (S3/Azure/GCS; Data Lake/Elastic/SQL) e credenciais válidas.

## Entradas
- `project_id`, `destinations[]` (ex.: `data_lake`, `elastic`, `sql`), `formats[]` (ex.: `parquet`, `jsonl`, `csv`), `slices[]` (ex.: `entities`, `relations`, `claims`, `communities`, `community_reports`, `text_units`, `insights`, `metrics`).
- Filtros: `tags[]`, `setor`, `tema`, `lang`, `sensitivity<=threshold`, `date_range` (incremental).
- Parâmetros: `incremental=true|false`, `partition_by={project_id,lang,month}`, `compression={snappy|gzip}`, `overwrite=false`, `max_rows_per_file`.

## Fluxo (alto nível)
1) Validar escopo e políticas; resolver destino(s) e credenciais por `tenant/project`.
2) Selecionar slices e construir consultas/extrações em lotes com paginação.
3) Aplicar filtros e políticas (mascaramento/redação de PII; exclusão por `sensitivity`).
4) Serializar em formatos escolhidos; gravar em `object storage`/`data lake` com particionamento.
5) Opcional: publicar índices/visões em Elastic/SQL (para busca/BI).
6) Registrar `ExportJob` (status, contagens, custos, localização dos artefatos) e emitir eventos de domínio.

- Nós SKGraph (exemplos): `FunctionGraphNode(ExportService.Prepare)`, `ForeachLoopGraphNode(Slices)`, `ResultAggregator`, `RetryPolicyGraphNode`, `PIIFilter`.

## Regras/Políticas e Guardrails
- RBAC/ABAC por `tenant/project/tag/setor/tema` e por sensibilidade de `TextUnit`.
- Direito ao esquecimento: excluir registros por `docId/unitId/entityId` antes de exportar; respeitar tombstones.
- Idempotência: `ExportJob` com chave `{projectId, slice, date_range, partition}`.
- Orçamento: limites por job (`max_rows`, `max_files`, `max_runtime`); retries idempotentes e timeouts.

## Saídas e Artefatos
- Arquivos em `artifacts/exports/{project}/{slice}/{partition}/file.*` com metadados (`_SUCCESS`, manifest).
- Tabelas/índices opcionais: Elastic (`index_name`), SQL (`schema.table`), catálogos do Data Lake (Glue/Metastore).
- `ExportJob` (SQL): `{ job_id, project_id, params, status, counts, cost, output_refs[], started_at, finished_at }`.

## Parâmetros e Ajustes
- `incremental` com base em `last_sync`/`updated_at` por slice.
- `partition_by` por mês/idioma/projeto para eficiência de leitura em BI.
- `formats` e `compression` compatíveis com o ecossistema alvo.
- `mask_pii`, `sensitivity_threshold`, `deduplicate=true`.

## Métricas e Critérios de Aceite
- Métricas: tempo total, custo (tokens/IO), throughput (linhas/s), tamanho por arquivo, taxa de rejeição por política.
- Aceite:
  - Exportações reproduzíveis com mesmos parâmetros e partições.
  - Conformidade com políticas de sensibilidade/PII e direito ao esquecimento.
  - Catálogo/manifest disponível e legível por jobs de BI.

## Rastreabilidade
- Roadmap: seções 11 e 16 em `docs/roadmap.md` (BI integrado, exportações, dashboards).
- Roadmap técnico: seções 3.1, 7, 9, 10 e 11 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Custo e latência em projetos grandes → usar incremental, particionamento e compressão.
- Divergência de esquemas entre versões → versionar DTOs e manter compatibilidade.
- Vazamento de PII/sensível → reforçar filtros, auditoria e revisões de acesso.

## Exemplos
- Criar job de exportação (exemplo de contrato):
```
POST /v1/exports
{
  "project_id": "proj-123",
  "destinations": ["data_lake"],
  "formats": ["parquet"],
  "slices": ["entities", "claims", "community_reports"],
  "filters": { "lang": "pt", "sensitivity": "<=Interno" },
  "params": { "incremental": true, "partition_by": ["project_id", "month"], "compression": "snappy" }
}
```
- Resposta do job:
```
{
  "job_id": "exp-001",
  "status": "running",
  "output_refs": [],
  "started_at": "2025-09-20T12:00:00Z"
}
```
- Consulta de status:
```
GET /v1/jobs/exp-001
{
  "job_id": "exp-001",
  "status": "succeeded",
  "counts": { "entities": 12034, "claims": 5840 },
  "output_refs": ["s3://bucket/artifacts/exports/proj-123/entities/2025-09/part-000.parquet"],
  "finished_at": "2025-09-20T12:12:32Z"
}
```

## Variantes e Extensões
- Publicação em Elastic/OpenSearch com mapeamentos otimizados para busca.
- Materializações em SQL para KPIs (views/CTEs) e camadas semânticas para BI.
- Gatilhos agendados (cron) e eventos de domínio após ingestão/atualização de grafo.
