# Semantic Kernel Graph RAG

Plataforma de RAG híbrido para empresas, construída em .NET 8 e orquestrada com SKGraph, que combina Vector Search, GraphRAG (local e global) e DRIFT para entregar respostas confiáveis, citáveis e governáveis — com custo previsível e observabilidade de ponta a ponta.

## Visão Executiva

Organizações acumulam conteúdos em múltiplos formatos, idiomas e sistemas. Este projeto conecta tudo isso em uma camada de conhecimento navegável por grafos, com consultas que alternam de forma inteligente entre buscas vetoriais, expansão local por entidades e visão global por comunidades — sempre com controles de segurança e trilhas de auditoria.

O resultado é uma plataforma pronta para ambientes corporativos: confiável, explicável, segura e eficiente em custos.

## O que entregamos

- Respostas com citações precisas às evidências (página, seção, tempo em áudio/vídeo, bbox em imagens)
- Panorama global por comunidades com drill-down para detalhes locais
- Roteamento automático por intenção (Vector, Local GraphRAG, Global GraphRAG, DRIFT)
- Governança (RBAC/ABAC), privacidade (PII) e direito ao esquecimento
- Observabilidade completa (latência, custo, tokens, modelos) e métricas de qualidade
- Operação multi-tenant, escalável e com orçamento por consulta

## Capacidades Principais

- Ingestão e Indexação
  - Upload/ZIP/GDrive, normalização e fragmentação híbrida (heading-aware, semântica, janelas)
  - Embeddings + BM25 com filtros por tags/setor/tema/idioma

- Extração e Grafo de Conhecimento
  - IE estruturada (entidades, relações, afirmações) com evidências por unidade de texto
  - Fusão com aliases/chaves de bloqueio; resumos por entidade em múltiplos níveis

- Comunidades e Relatórios
  - Detecção hierárquica (Leiden/Louvain) e relatórios executivos por comunidade (níveis)
  - Representações para busca global focada em consulta (QFS)

- Consulta e Roteamento por Intenção
  - Vector RAG (BM25 ∪ ANN + re-rank)
  - Local GraphRAG (expansão k-hop por entidades com citações)
  - Global GraphRAG (QFS via relatórios de comunidade)
  - DRIFT (global → local com reclassificação final)
  - Parâmetros de custo e seleção dinâmica de contexto

- Segurança, Privacidade e Governança
  - Classificação de sensibilidade (Público/Interno/Confidencial), redação/mascaramento de PII
  - RBAC/ABAC por tenant/projeto/taxonomia e auditoria de consultas

- Observabilidade, Qualidade e A/B
  - Traços por nó do grafo de execução (latência, custo, tokens, provider/model)
  - Métricas de relevância, cobertura, fundamentação e diversidade; testes A/B por modo/intent

- Auto Few-Shots e Prompts
  - Geração, curadoria e versionamento de exemplos por projeto/idioma
  - Evolução contínua a partir de Q/A validados e seus traces

- Insights e BI
  - Playbooks (riscos, tendências, FAQs, conformidade) e exportação para Data Lake/BI

- Diferenciais Multimodais e XAI
  - Citações multimodais contextuais (áudio/vídeo, imagem, documento)
  - Consultas cross-modais e trace XAI com subgrafo serializado

Consulte detalhes técnicos e decisões de arquitetura em `docs/roadmap.md` e `docs/roadmap_tecnico.md`. Os cenários estão mapeados em `docs/usecases/index.md` e suas execuções em grafos em `docs/usegraphs/index.md`.

## Arquitetura em Alto Nível

- Gateway de API (REST/JSON; opcional gRPC) com autenticação e autorização
- Núcleo de Serviço: pipelines como grafos (SKGraph) com roteamento por intenção
- Workers de Ingestão: execução assíncrona e incremental por fonte
- Camada de Conhecimento: objetos, vetorial, BM25, grafo (Neo4j) e relacional
- Observabilidade: logs estruturados, métricas e traços correlacionados por consulta/projeto/usuário
- Console Administrativo: gestão de tenants, taxonomias, prompts, políticas e recursos

## Roadmap

- Fase 1 – MVP: ingestão, fragmentação, embeddings, BM25 e consulta vetorial, com taxonomias, autenticação e traços básicos
- Fase 2 – Grafo e Local: IE + consolidação em grafo; consultas locais k-hop com citações
- Fase 3 – Global e Híbrido: comunidades, relatórios por nível, busca global (QFS) e fluxo DRIFT
- Fase 4 – Auto-Tuning e Insights: few-shots automáticos, playbooks e otimizações avançadas de custos/cache

## Para quem é

- Equipes de Dados/IA que precisam acelerar RAG corporativo, com governança
- Times de Produto que buscam respostas citáveis e explicáveis para usuários finais
- Áreas reguladas (Jurídico, Compliance, Saúde, Financeiro) que exigem auditabilidade

## Como começar

- Documentação e exemplos do SKGraph:
  - NuGet: [SemanticKernel.Graph](https://www.nuget.org/packages/SemanticKernel.Graph)
  - Documentação: [skgraph.dev](https://skgraph.dev/)
  - Exemplos: [semantic-kernel-graph-docs/examples](https://github.com/kallebelins/semantic-kernel-graph-docs/tree/main/examples)

- Documentos deste repositório:
  - Roadmap: `docs/roadmap.md`
  - Roadmap técnico: `docs/roadmap_tecnico.md`
  - Usecases: `docs/usecases/index.md`
  - GraphCases: `docs/usegraphs/index.md`

## Comunidade e Contribuições

Queremos construir isso em conjunto. Participe:

- Abra issues com ideias, dúvidas, bugs ou propostas de melhoria
- Sugira RFCs para funcionalidades relevantes ao ecossistema corporativo
- Envie PRs focados, com descrição clara do problema, abordagem e impacto
- Mantenha alinhamento com os padrões do `SemanticKernel.Graph` (arquitetura, nomenclatura, contratos)

Se você ou sua organização desejam coevoluir módulos (conectores, storage, grafos sociais, XAI, BI), vamos conversar nas issues para coordenar o roadmap.

## Licença

Licença: Apache-2.0 com Commons Clause (no-sale). Consulte `LICENSE`.

Permissões: estudo, uso interno (inclusive corporativo), modificação e compartilhamento.

Restrições: proibida a venda do software/serviço cujo valor derive substancialmente desta funcionalidade. Licenças comerciais podem ser oferecidas à parte.
