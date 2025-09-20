# Usecase: Roteamento por intenção (heurísticas e/ou classificadores)

## Contexto e Problema
Escolher o modo de consulta correto (Vector, Local, Global, DRIFT) impacta diretamente custo, latência e qualidade. Sem roteamento, o sistema pode usar estratégias subótimas. Precisamos detectar intenção e aplicar regras/classificadores para direcionar a pergunta.

## Objetivo
Classificar a intenção da pergunta e selecionar o modo apropriado, com confiança e fallback; registrar traços da decisão para auditoria e A/B.

## Atores e Escopo
- Usuário (via `POST /v1/query`, `mode=auto`)
- Serviços: `IntentClassifier`, `SwitchGraphNode`/`ConditionalGraphNode`, `Guardrails`, `ABTester`
- Dentro: extração de features, classificação, heurísticas e seleção de aresta no DAG
- Fora: execução dos modos (definidos em seus próprios usecases)

## Pré-condições
- `intent_schemas` definidos por projeto: intents, padrões, classificadores e roteador
- Few-shots de intenção opcionais por idioma/setor/tema

## Entradas
- `{ project_id, question, mode=auto, filters {...}, params {...} }`
- Sinais: tamanho da pergunta, presença de termos-chave ("panorama", "comparar", nomes de entidades), estrutura (pergunta curta vs narrativa)

## Fluxo (alto nível)
- Etapa 1: Extrair sinais/feições; aplicar regras rápidas
- Etapa 2: Classificador (zero-shot/few-shot) produz distribuição sobre `{Vector, Local, Global, DRIFT}` e intents de domínio
- Etapa 3: Selecionar aresta no DAG: `DetectIntent` → `{ VectorRag | LocalGraph | GlobalQFS | DRIFT }`
- Etapa 4: Se confiança baixa, fallback conservador (ex.: DRIFT) ou pedir esclarecimento (opcional)
- Nós SKGraph: `DetectIntent` com `SwitchGraphNode` e arestas condicionais

## Regras/Políticas e Guardrails
- Heurísticas exemplares:
  - "panorama", "tendência", "mapa" → GlobalQFS
  - pergunta curta com termo específico → Vector
  - entidades/pessoas/sistemas/regulação → LocalGraph
  - longa/ambígua → DRIFT
- ABAC/RBAC: negar intents desabilitados no projeto; logs para auditoria

## Saídas e Artefatos
- `intent`: `{mode, confidence, rationale}`
- `trace`: features/sinais, decisão, alternativas consideradas
- `metrics`: acurácia de roteamento por classe (A/B), impacto em latência/custo

## Parâmetros e Ajustes
- `threshold_confidence` (ex.: 0.55–0.7)
- Pesos de heurística vs classificador; fallback prioritário
- Feature flags para intents habilitados por projeto

## Métricas e Critérios de Aceite
- Aumento de satisfação/qualidade vs escolha fixa de modo
- Redução de custo médio por consulta mantendo groundedness
- Taxa de quedas para fallback dentro de faixa aceitável (<10%)

## Rastreabilidade
- Roadmap: seções 5 (Roteador de Intenções), 7/10 (Observabilidade/A/B)
- Roadmap técnico: seções 1.2 (Switch/Conditional nodes), 5 (Modos), 7 (A/B)

## Riscos e Considerações
- Overfitting de heurísticas: validar com A/B e atualizar few-shots
- Ambiguidade linguística: pedir esclarecimento quando orçamento permite
- Lidar com multilíngue e jargão setorial: perfis por projeto/idioma

## Exemplos
- Pergunta: "Faça um panorama de riscos em Segurança da Informação"
  - `intent` → `global_graph` (confiança 0.78)

## Variantes e Extensões
- Ensemble de classificadores e regras; roteamento adaptativo por custo/latência corrente
