# Mapa de Usecases vs Roadmap

> Use o `_template.md` para detalhar cada usecase. Este índice mapeia as áreas do roadmap em arquivos de usecase a serem preenchidos.

Veja também: [Índice de GraphCases (UC ↔ GC)](../usegraphs/index.md)

## Ingestão e Indexação
- [Upload/ZIP/GDrive → Normalize/Chunk](./ingestao-e-indexacao/uc01_ingestao_indexacao_upload_normalize_chunk.md)
- [Embeddings + BM25 (filtros por tags/setor/tema/lang)](./ingestao-e-indexacao/uc02_ingestao_indexacao_embeddings_bm25.md)
- [Classificação de sensibilidade e políticas de privacidade](./ingestao-e-indexacao/uc03_ingestao_indexacao_classificacao_sensibilidade_politicas.md)

## Extração e Grafo
- [IE estruturada (entities, relations, claims) citando `unit_id`](./extracao-e-grafo/uc04_extracao_grafo_ie_estruturada_entities_relations_claims.md)
- [Fusão/Merge no grafo (aliases, chaves de bloqueio, evidências)](./extracao-e-grafo/uc05_extracao_grafo_fusao_merge_grafo.md)
- [Consulta local k-hop com evidências citadas](./extracao-e-grafo/uc06_extracao_grafo_consulta_local_khop_evidencias.md)

## Comunidades e Relatórios
- [Detecção hierárquica (Leiden/Louvain) por nível](./comunidades-e-relatorios/uc07_comunidades_relatorios_deteccao_hierarquica.md)
- [Community Reports e embeddings para QFS](./comunidades-e-relatorios/uc08_comunidades_relatorios_community_reports_embeddings_qfs.md)

## Consulta
- [Vector RAG (BM25 ∪ ANN com re-rank)](./consulta/uc09_consulta_vector_rag_bm25_ann_rerank.md)
- [Local GraphRAG (expansão por entidades)](./consulta/uc10_consulta_local_graphrag_expansao_entidades.md)
- [Global GraphRAG (QFS por comunidades)](./consulta/uc11_consulta_global_graphrag_qfs.md)
- [DRIFT (global → local com reclassificação)](./consulta/uc12_consulta_drift_hibrido_global_para_local.md)
- [Roteamento por intenção (heurísticas e/ou classificadores)](./consulta/uc13_consulta_roteamento_por_intencao.md)
- [Parâmetros e filtros](./consulta/uc14_consulta_parametros_e_filtros.md)

## Observabilidade e Governança
- [Traços por nó (latência, custo, tokens, provider/model)](./observabilidade-e-governanca/uc15_observabilidade_governanca_tracos_por_no.md)
- [RBAC/ABAC, redação/mascaramento de PII, direito ao esquecimento](./observabilidade-e-governanca/uc16_observabilidade_governanca_rbac_abac_pii_direito_esquecimento.md)

## Multimodais e XAI
- [Citações multimodais: `time_range`, `bbox`, `page/section/heading`](./multimodais-e-xai/uc17_multimodais_xai_citacoes_multimodais.md)
- [Consultas cross-modais e ordenação unificada](./multimodais-e-xai/uc18_multimodais_xai_consultas_cross_modais_ordenacao_unificada.md)
- [Trace XAI: serialização de subgrafo/mermaid/Neo4j Bloom/GraphView](./multimodais-e-xai/uc19_multimodais_xai_trace_xai_serializacao_subgrafo.md)

## Insights e BI
- [Playbooks: riscos, tendências, FAQ, matriz de conformidade](./insights-e-bi/uc20_insights_bi_playbooks_riscos_tendencias_faq_matriz_conformidade.md)
- [Exportações para Data Lake/BI e dashboards](./insights-e-bi/uc21_insights_bi_exportacoes_datalake_bi_dashboards.md)

## Operação e Custos
- [Orçamento por consulta e seleção dinâmica de contexto](./operacao-e-custos/uc22_operacao_custos_orcamento_por_consulta_selecao_dinamica.md)
- [Cache multicamadas](./operacao-e-custos/uc23_operacao_custos_cache_multicamadas.md)
- [Indexação incremental e reprocesso seletivo](./operacao-e-custos/uc24_operacao_custos_indexacao_incremental_reprocesso_seletivo.md)

## Integração Social/Organizacional
- [Pessoas/Equipes/Projetos e relações com conteúdos](./integracao-social-organizacional/uc25_integracao_social_organizacional_pessoas_equipes_projetos_relacoes_conteudos.md)

## Auto Few-Shots
- [Geração, curadoria e versionamento por projeto/idioma](./auto-few-shots/uc26_auto_fewshots_geracao_curadoria_versionamento.md)
