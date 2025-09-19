# Características e Soluções Técnicas

Este documento consolida as características essenciais e descreve soluções técnicas detalhadas para a plataforma de RAG híbrido (Vector + GraphRAG + DRIFT) baseada em .NET 8 e no ambiente de grafos corporativos. O objetivo é oferecer um guia completo, cobrindo desde ingestão e indexação até consulta, segurança e avaliação.

---

## 1) Princípios e Diretrizes

- **IE estruturada**: Extração de entidades, relações e afirmações a partir de trechos de texto, com esquema padronizado por domínio e citação explícita das unidades de evidência. A normalização tipológica (por exemplo, tipos padronizados de entidade) e a presença de metadados são mandatórias.
- **Grafo de conhecimento**: Consolidação de entidades e arestas com resolução de aliases, fusão por chaves de bloqueio e manutenção de evidências. Suporta resumos por entidade em múltiplos níveis para navegação global.
- **Comunidades hierárquicas**: Detecção de módulos em múltiplos níveis (do fino ao macro) para permitir navegação por agrupamentos semânticos e sumarização bottom-up por comunidade.
- **Modos de consulta complementares**: Operação em três eixos principais: local (expansão curta no grafo), global (resumo focado em consulta) e híbrido (global → local com reclassificação), com roteamento por intenção.
- **Seleção dinâmica orientada a custo**: Parâmetros que limitam tokens e profundidade. Abertura incremental de comunidades e contexto apenas quando o ganho esperado supera o custo.
- **Observabilidade e governança**: Traços por etapa, métricas de qualidade e custos, com políticas de segurança, privacidade e auditoria por projeto/tenant.

---

## 2) Arquitetura em Camadas

- **Gateway de API**: Interface REST/JSON (e opcionalmente gRPC) com autenticação e autorização baseadas em padrões de mercado. Responsável por receber uploads, iniciar sincronizações, expor consultas e insights.
- **Núcleo de Serviço**: Orquestração de pipelines como grafos dirigidos acíclicos, com nós bem definidos para cada responsabilidade (detectar, normalizar, fragmentar, indexar, extrair, consolidar, detectar comunidades, sumarizar, classificar intenção e responder).
- **Workers de Ingestão**: Execução assíncrona com checkpointing e reprocessamento incremental por fonte. Permitem escalar volume e isolar workloads de ingestão.
- **Camada de Conhecimento**:
  - Armazenamento de objetos para originais e artefatos (relatórios, sumários, auditorias).
  - Armazenamento vetorial para embeddings de trechos e relatórios.
  - Mecanismo de busca lexical (BM25) para recuperação rápida e filtrável.
  - Banco de grafo para entidades, relações, comunidades e relatórios.
  - Banco relacional para metadados, configurações, auditoria e métricas.
- **Observabilidade**: Coleta de logs estruturados, métricas e traços por nó do grafo de execução, com correlação por consulta, projeto e usuário.
- **Console Administrativo**: Gestão de tenants, taxonomias, prompts, políticas e recursos.

---

## 3) Modelo de Dados Essencial

- **Projeto**: Identidade do projeto, tenant, taxonomia (tags, setor, tema), políticas (privacidade, retenção), modos RAG habilitados.
- **Fonte**: Registro de origem dos documentos (upload, armazenamento externo, repositórios), estado e última sincronização.
- **Documento**: Metadados de arquivo e classificação herdada (tags, setor, tema, idioma), status e integridade.
- **Unidade de Texto (Chunk)**: Trecho com ordem, texto, contagem de tokens, metadados estruturais (título, seção, página) e relacionamento com índices.
- **Grafo**: Entidades (com aliases e resumos por nível), relações (incluindo evidência), afirmações (texto e evidência), comunidades (membros, nível, entidades centrais, sumário).
- **Índices**: Entradas vetoriais para trechos e relatórios; postings lexicais para BM25.
- **Config NLP**: Perfis de prompt, few-shots por tarefa, esquemas de intenção e roteadores.
- **Consulta e Insight**: Registros de consultas (texto, modo, parâmetros, custos, latência) e resultados derivados (playbooks, tendências, lacunas, FAQs).

---

## 4) Pipeline de Ingestão (DAG)

- **Detecção e Normalização**: Identifica idioma, extrai texto de múltiplos formatos e preserva dicas estruturais como títulos e seções para orientar fragmentação.
- **Taxonomia e Metadados**: Atribuição por regras e, quando necessário, por classificadores, com herança de atributos do documento para as unidades de texto.
- **Fragmentação (Chunking)**: Estratégias híbridas que combinam limites por headings, divisão semântica e janelas sobrepostas. Opção para preservar blocos indivisíveis como tabelas e trechos de código.
- **Embeddings e BM25**: Geração de representações vetoriais e indexação lexical com metadados ricos para filtros rápidos.
- **Extração de IE**: Geração de estruturas de entidades, relações e afirmações em formato normalizado, com segunda passada de reforço em unidades de alta densidade informacional.
- **Consolidação de Grafo**: Fusão por aliases e chaves de bloqueio, evitando explosão combinatória; manutenção de evidências por unidade e atualização incremental do subgrafo do projeto.
- **Detecção de Comunidades**: Particionamento em níveis, possibilitando visão hierárquica do conhecimento.
- **Relatórios de Comunidade**: Sumarização bottom-up por comunidade, com seções orientadas a objetivos, riscos, processos, dependências, principais pontos e lacunas. Armazenamento de representações para busca global focada em consulta.
- **Auto-Tuning (Opcional)**: Geração assistida de exemplos por tarefa (extração, classificação, resposta, relatório), com validações de balanceamento, deduplicação e segurança.
- **Qualidade e Guardrails**: Classificação de sensibilidade por trecho, aplicação de políticas de privacidade, cálculo de métricas de cobertura e densidade.

---

## 5) Modos de Consulta e Roteamento por Intenção

- **Vector RAG**: Combinação de recuperação lexical e vetorial para perguntas objetivas ou com termos específicos. Utiliza reclassificação final para priorizar evidências mais relevantes.
- **GraphRAG Local**: Expansão direcionada por entidades em poucos saltos para coletar evidências estruturadas, com síntese de resposta citável.
- **GraphRAG Global (QFS)**: Seleção de comunidades relevantes, navegação da hierarquia e uso de relatórios de comunidade em um fluxo de map-reduce para produzir visão panorâmica.
- **DRIFT Híbrido**: Pré-filtragem global para reduzir o espaço de busca, seguida por sub-perguntas locais e reclassificação final.
- **Roteador de Intenções**: Classificação por regras e/ou modelos para direcionar perguntas ao modo apropriado com base no vocabulário, tamanho e intenção (ex.: panorama, procedimento, compliance, comparação).
- **Seleção Dinâmica**: Parâmetros de orçamento (número de comunidades, largura de feixe, profundidade, tokens de contexto) controlam a abertura progressiva de contexto.

---

## 6) Estratégias de Fragmentação e Indexação

- **Heading-aware**: Respeito a hierarquias de títulos (níveis 1 a 4) para preservar coerência temática.
- **Semantic split**: Delimitação por coesão semântica via sinais linguísticos e embedding de sentenças.
- **Table/Code aware**: Tratamento especial para tabelas e trechos técnicos, preservando contexto adjacente.
- **Rolling window híbrida**: Janelas sobrepostas para continuidade e redução de cortes duros.
- **Configuração por Projeto**: Parâmetros como tamanho máximo, overlap, tamanho mínimo de sentença e preservação de listas/tabelas.
- **Indexação Vetorial**: Coleções por tenant e projeto, com payloads de metadados (documento, unidade, taxonomia e idioma).
- **BM25**: Índice lexical com campos facetados para filtros e recuperação rápida.

---

## 7) Extração de Informação (IE) e Pós-Processo

- **Esquemas Normalizados**: Definição de tipos de entidade e relação por domínio, formatos de saída consistentes e citação obrigatória das unidades de evidência.
- **Alias e Normalização**: Case folding, lematização e regras específicas por tipo para unificar referências a uma mesma entidade.
- **Chaves de Bloqueio**: Combinações de atributos para reduzir comparações desnecessárias ao mesclar entidades e relações.
- **Passada de Gleanings**: Segunda rodada seletiva em unidades densas para aumentar o recall sem elevar custos de forma indiscriminada.

---

## 8) Comunidades e Relatórios

- **Detecção Hierárquica**: Partições por nível, do detalhado ao macro, permitindo panoramas e drill-down progressivo.
- **Métricas de Particionamento**: Avaliação por modularidade e qualidade de corte; opcionalmente, refinamentos locais.
- **Relatórios de Comunidade**: Sumários executivos com objetivos, riscos, processos, dependências, principais pontos e lacunas, vinculados a evidências e a entidades centrais do grupo.
- **Representações para QFS**: Embeddings de relatórios para busca global e seleção por relevância.

---

## 9) Segurança, Privacidade e Governança

- **Classificação de Sensibilidade**: Trechos rotulados como Público, Interno ou Confidencial, influenciando políticas de exibição e resposta.
- **Redação e Mascaramento**: Aplicação de políticas de privacidade para remoção ou mascaramento de informações pessoais em contextos públicos.
- **Citações Obrigatórias**: Para respostas críticas, exigência de referência a unidades de evidência ou comunidades.
- **Controle de Acesso**: Políticas de acesso por tenant, projeto e taxonomia (tags, setor, tema), com auditoria de consultas e resultados.
- **Direito ao Esquecimento**: Mecanismos para remover ou reter seletivamente documentos, unidades ou entidades.

---

## 10) Observabilidade, Qualidade e Avaliação

- **Traços por Etapa**: Cada nó do pipeline registra latência, custo e utilização de tokens, com correlação por consulta.
- **Métricas de Qualidade**: Indicadores de relevância, cobertura, fundamentação, diversidade e empoderamento.
- **Benchmarks Internos**: Conjuntos de perguntas estratificados por classes (fato local, procedimento, comparativo, panorama global, compliance) com verificação automatizada.
- **A/B por Modo e Intent**: Comparação controlada entre estratégias (Vector, Local, Global, DRIFT) e ajustes de roteamento.
- **Monitor de Drift**: Rastreamento de mudança de vocabulário por setor/tema e ajustes de prompts e estratégias.

---

## 11) Operação, Custos e Escala

- **Orçamento por Consulta**: Limites rígidos de tokens e passos por execução.
- **Seleção Dinâmica de Contexto**: Abertura incremental de comunidades e níveis conforme ganho esperado.
- **Cache Multicamadas**: Armazenamento de respostas, relatórios e recursos de reclassificação para reduzir latência e custo recorrentes.
- **Indexação Incremental**: Processamento de diffs por origem e reprocessamento seletivo após alterações.
- **Multi-tenant e Isolamento**: Separação por tenant e projeto em índices, grafos e armazenamento, com quotas e SLOs configuráveis.

---

## 12) Auto Few-Shot Tuning e Prompts

- **Amostragem Estratificada**: Seleção de exemplos por setor, tema e tags para cobrir o espaço de domínio.
- **Geração de Exemplos**: Conjuntos para extração, classificação, resposta e sumarização com estilo corporativo, em idioma do projeto.
- **Validação e Higiene**: Balanceamento por classe, deduplicação, escaneamento de toxicidade e PII.
- **Versionamento por Projeto**: Armazenamento e evolução controlada dos exemplos por idioma e domínio.

---

## 13) API e Contratos

- **Ingestão**: Criação de projetos, registro de fontes, upload e sincronização, com acompanhamento de jobs.
- **Configuração**: Definição de taxonomias, modos e limites de custo, perfis de prompt e esquemas de intenção.
- **Consulta**: Envio de perguntas com filtros e parâmetros; retorno com resposta, citações, traços e eventuais insights.
- **Insights**: Execução de playbooks como riscos por setor, tendências e FAQs; obtenção de resultados por projeto e tipo.
- **Observabilidade**: Acesso a traços de consultas e métricas agregadas de desempenho e custo.

---

## 14) Roadmap de Entregas

- **Fase 1 – MVP**: Ingestão, fragmentação, embeddings, BM25 e consulta vetorial, com taxonomias, autenticação e traços básicos.
- **Fase 2 – Grafo e Local**: Extração de informação e consolidação em grafo; consultas locais com expansão de entidades e citações estruturadas.
- **Fase 3 – Global e Híbrido**: Detecção hierárquica de comunidades, relatórios por nível, busca global focada em consulta e fluxo híbrido.
- **Fase 4 – Auto-Tuning e Insights**: Geração automática de exemplos, playbooks de insights e otimizações avançadas de custos e cache.

---

## 15) Critérios de Aceite

- Documentação cobre ingestão, indexação, IE, grafo, comunidades, consultas, segurança, observabilidade, avaliação e custos.
- Conceitos e soluções descritos de forma aplicável, com termos consistentes e alinhados à arquitetura corporativa.
- Itens de governança e privacidade contemplados, incluindo citações obrigatórias e direito ao esquecimento.
- Alinhamento com o roadmap e com os modos de operação previstos (Vector, Local, Global, DRIFT).

---

## 16) Diferenciais Disruptivos

- **Citações multimodais contextuais**: Citações precisas por mídia (áudio/vídeo com faixa temporal e diarização, imagem com `bbox` OCR, documento com página/seção/heading), com metadados padronizados.
  - **Implementável**: Propagar `unit_id`, `time_range`, `bbox`, `page`, `section` do pipeline de ingestão até a resposta e UI; normalizar formatos por tipo de mídia; persistir nas unidades e nas estruturas de citação de respostas e relatórios.
  - **Benefício**: Aumenta confiança e auditabilidade ao apontar a evidência exata.

- **Consultas cross-modais**: Permite perguntas que combinam fala, imagem e texto, retornando resposta única composta por evidências heterogêneas.
  - **Implementável**: Incluir intents multimodais no roteador (ex.: `Intent.CrossModal`); adaptar prompts para fundir citações de mídias distintas; unificar reclassificação e ordenação final; assegurar filtros por taxonomia e sensibilidade por mídia.
  - **Benefício**: Respostas mais ricas e aderentes a consultas naturais do usuário.

- **RAG explicável (XAI + grafo)**: Além da resposta, expõe o raciocínio como um traço navegável: comunidades visitadas, entidades centrais, claims e evidências.
  - **Implementável**: Serializar o subgrafo/trace por consulta em JSON (nós, arestas, passos, métricas); expor via API; oferecer renderização em Mermaid/Neo4j Bloom/GraphView no console.
  - **Benefício**: Explicabilidade e conformidade em ambientes regulados.

- **Agente proativo com detecção de eventos**: Watchers contínuos que monitoram o grafo/conteúdos e disparam alertas automáticos quando padrões são detectados.
  - **Implementável**: Jobs agendados com classificadores de evento e janelas de tempo; queries contínuas no grafo; publicação em canais (Teams/Slack/Webhook) com citações.
  - **Benefício**: Do uso consultivo ao proativo, reduzindo tempo de reação a mudanças relevantes.

- **Treinamento dinâmico de few-shots (auto evolução)**: Reaproveita Q/A validados e seus traces para evoluir exemplos de prompts por projeto/idioma.
  - **Implementável**: Persistir Q/A + trace; job de curadoria para balanceamento, deduplicação e higiene; versionar e aplicar automaticamente `PromptProfiles` por projeto.
  - **Benefício**: Adaptação contínua ao jargão e às práticas da organização.

- **Integração com grafos de pessoas/processos**: Camada social/organizacional (Pessoas, Equipes, Projetos) a partir de participações e menções em documentos/áudio/vídeo.
  - **Implementável**: Novos tipos de entidade e relação (ex.: "participou de", "aprovou", "discutiu"); mapeamento para departamentos e processos; consultas por envolvimento e impacto.
  - **Benefício**: Perguntas organizacionais e análises de colaboração e responsabilidade.

- **Governança adaptativa**: Respostas e citações condicionadas dinamicamente ao perfil (RBAC/ABAC) e à sensibilidade do conteúdo.
  - **Implementável**: Aplicar políticas no momento de montagem de prompts e de exibição; redigir/mascarar conforme classe de sensibilidade; auditoria por usuário, consulta e resultado.
  - **Benefício**: Conformidade com LGPD e políticas internas sem sacrificar utilidade.

- **RAG + métricas de negócio (BI integrado)**: Exportação de entidades/claims e indicadores para datalake/BI e execução de playbooks de insights.
  - **Implementável**: Jobs de exportação para Data Lake/Elastic; dashboards em PowerBI/Tableau; playbooks SKGraph executando consultas temáticas (tendências, riscos, FAQs) por projeto.
  - **Benefício**: Geração contínua de inteligência executiva a partir do conhecimento corporativo multimodal.