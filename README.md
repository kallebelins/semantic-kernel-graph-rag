# Semantic Kernel Graph RAG

Plataforma de RAG híbrido (Vector + GraphRAG + DRIFT) construída em .NET 8, integrada ao seu projeto de grafos `SemanticKernel.Graph`. Este repositório traz a API, orquestração e documentação de uso, enquanto a construção e consulta ao grafo são delegadas à biblioteca `SemanticKernel.Graph`.

> Para fundamentos de arquitetura, dados e APIs, veja `docs/roadmap.md` e `docs/roadmap_tecnico.md`.


## Visão geral

Este projeto habilita:

- Ingestão de documentos (upload/ZIP/GDrive/Git/HTTP) com detecção de idioma e normalização
- Chunking semântico e baseado em headings, embeddings e índice BM25
- Extração de entidades/relacionamentos/claims e montagem de grafo (via `SemanticKernel.Graph`)
- Comunidades (Leiden) e geração de community reports
- Modos de consulta: Vector RAG, Local GraphRAG, Global GraphRAG (QFS) e DRIFT híbrido
- Guardrails (PII/LGPD), auditoria, traços e métricas

Arquitetura de alto nível (resumo): API (.NET 8) → Serviços (Plugins SK) → Armazenamentos (Vector, BM25, Graph, SQL, Object Storage). Detalhes em `docs/roadmap.md`.


## Requisitos

- .NET SDK 8.0+
- Docker (para stores locais)
- Opcional, conforme modo:
  - Vector Store: Qdrant (recomendado) ou pgvector/Weaviate
  - Graph DB: Neo4j
  - BM25: OpenSearch/Elasticsearch (opcional; BM25 pode ser desabilitado no MVP)
  - SQL: Postgres/SQL Server para metadados
  - Object Storage: MinIO/S3 (para artefatos)


## Subindo dependências locais (Docker)

Compose mínimo (Vector + Graph). Salve como `docker-compose.yml` na raiz e rode `docker compose up -d`.

```yaml
services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_data:/qdrant/storage

  neo4j:
    image: neo4j:5
    environment:
      - NEO4J_AUTH=neo4j/neo4j
      - NEO4J_server_memory_pagecache_size=1G
    ports:
      - "7474:7474"
      - "7687:7687"
    volumes:
      - neo4j_data:/data

volumes:
  qdrant_data:
  neo4j_data:
```


## Configuração

Crie um arquivo `.env` na raiz com as variáveis relevantes (exemplo):

```dotenv
# Ambiente
ASPNETCORE_ENVIRONMENT=Development

# LLM Providers (ajuste conforme seu provedor)
OPENAI__API_KEY=
AZURE_OPENAI__Endpoint=
AZURE_OPENAI__ApiKey=
AZURE_OPENAI__Deployment=

# Vector Store (Qdrant)
QDRANT__Url=http://localhost:6333

# Graph (Neo4j)
NEO4J__Uri=bolt://localhost:7687
NEO4J__Username=neo4j
NEO4J__Password=neo4j

# BM25 (opcional)
OPENSEARCH__Url=http://localhost:9200

# SQL (opcional)
POSTGRES__ConnectionString=Host=localhost;Port=5432;Database=skgraph;Username=postgres;Password=postgres

# Object Storage (opcional)
STORAGE__Endpoint=http://localhost:9000
STORAGE__Bucket=skgraph
STORAGE__AccessKey=
STORAGE__SecretKey=

# Auth (opcional)
AUTH__Authority=http://localhost:8080/realms/your-realm
AUTH__Audience=skgraph-api
```

Observação: O projeto usa a biblioteca `SemanticKernel.Graph` para operações de grafo. Mantenha-a disponível no seu ambiente de solução e siga os padrões de código e exemplos do seu repositório.


## Executando localmente (Windows PowerShell)

1) Suba as dependências:

```powershell
docker compose up -d
```

2) Restaure e execute o serviço/API (ajuste o caminho do projeto quando aplicável):

```powershell
dotnet restore
dotnet build -c Debug
dotnet run --project src/Service --no-launch-profile
```

Caso a solução use outro projeto de entrada, ajuste o caminho em `--project`.


## API principal (resumo)

Endpoints esperados (ver contratos detalhados em `docs/roadmap_tecnico.md`):

- POST `/v1/projects` – cria projeto/tenant
- POST `/v1/projects/{id}/sources` – registra origem
- POST `/v1/projects/{id}/upload` – upload de arquivo/ZIP
- POST `/v1/projects/{id}/sync` – sincroniza origem (gdrive/git/http)
- GET  `/v1/jobs/{job_id}` – status da indexação
- PUT  `/v1/projects/{id}/taxonomy` – configura taxonomia
- PUT  `/v1/projects/{id}/rag-config` – modos, chunking, limites de custo
- POST `/v1/query` – consulta (auto|vector|local_graph|global_graph|drift)
- POST `/v1/insights/generate` – executa playbooks

Exemplo de upload (ZIP):

```http
POST /v1/projects/abc/upload
Content-Type: multipart/form-data
file=@docs.zip
?tags=LGPD,Jurídico&setor=Compliance&tema=Políticas
```

Exemplo de consulta:

```json
{
  "project_id": "abc",
  "question": "Quais diretrizes LGPD para RH?",
  "mode": "auto",
  "filters": {"tags": ["LGPD"], "setor": "RH", "tema": "Políticas", "lang": "pt-BR"},
  "params": {"k": 12, "rerank": true, "max_tokens_ctx": 8000, "max_communities": 6}
}
```


## Integração com SemanticKernel.Graph

- Biblioteca: `https://www.nuget.org/packages/SemanticKernel.Graph`
- Exemplos: `https://github.com/kallebelins/semantic-kernel-graph-docs/tree/main/examples`
- Documentação: `https://skgraph.dev/`

Siga estritamente as diretrizes e padrões da biblioteca ao implementar criação, consulta, travessia e modificação de grafos neste projeto.


## Roadmap

Etapas e critérios detalhados em `docs/roadmap.md` (seções “Fase 1–4”) e especificações técnicas em `docs/roadmap_tecnico.md`.


## Contribuição

1) Abra uma issue descrevendo a mudança proposta
2) Siga o padrão de código da `SemanticKernel.Graph`
3) Adicione testes e documentação
4) Envie PR referenciando a issue


## Licença

Licença: Apache-2.0 com Commons Clause (no-sale). Consulte o arquivo `LICENSE` na raiz.

Permissões: estudo, uso interno (inclusive corporativo), modificação e compartilhamento.

Restrições: é proibido vender o software ou oferecê-lo como serviço pago cujo valor derive substancialmente de sua funcionalidade. O(s) autor(es) podem oferecer licenças comerciais separadas.


