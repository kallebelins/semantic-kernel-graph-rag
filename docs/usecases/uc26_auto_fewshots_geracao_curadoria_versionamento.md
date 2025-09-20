# Padrão de saída Usecase: Auto Few-Shots (Geração, Curadoria e Versionamento por Projeto/Idioma)

## Contexto e Problema
Prompts dependem de bons exemplos. Criar e manter few-shots manualmente é caro e não escala. É necessário gerar, curar e versionar exemplos automaticamente por projeto/idioma/tarefa (extraction, classification, answering, summarization), usando dados validados e traços do sistema.

## Objetivo
Automatizar a produção de few-shots de alta qualidade e seguros, alimentados por Q/A validados, traces e relatórios, com curadoria (balanceamento, dedup, higiene), sensíveis a idioma e domínio, e versionamento aplicável dinamicamente aos perfis de prompt.

## Atores e Escopo
- Usuários: Owners de projeto, analistas de NLP, administradores.
- Serviços: `InsightPlaybooks` (fonte de exemplos), `Trace/XAI` (coleta), `PIIFilter`, `PromptProfiles`, `AutoFewShotsService`.
- Fora do escopo: treinamento de modelos proprietários (foco é curadoria de exemplos para prompting).

## Pré-condições
- Registro de consultas e respostas com `trace` e citações.
- Métricas de qualidade por classe de pergunta.
- Políticas de PII e sensibilidade ativas.

## Entradas
- `project_id`, `task={extraction|classification|answering|summarization}`.
- Filtros: `lang`, `setor`, `tema`, `tags[]`, janela temporal.
- Parâmetros: `target_per_class`, `max_examples`, `dedup=true|false`, `toxicity_threshold`, `pii_mask=true|false`, `version_label`.

## Fluxo (alto nível)
1) Coletar candidatos: Q/A validados, trechos com boa cobertura/groundedness, relatórios de comunidade e seus traces.
2) Filtrar e higienizar: remover PII/sensível (ou mascarar), descartar baixa qualidade.
3) Deduplicar e balancear por classe/tema/idioma.
4) Normalizar formato por tarefa (ex.: extraction JSON `{entities,relations,claims}` com `unit_id`; answering com citações curtas; summarization estilo corporativo).
5) Versionar e publicar para `PromptProfiles` do projeto.
6) A/B opcional: comparar desempenho com e sem nova versão.

- Nós SKGraph (exemplos): `FunctionGraphNode(AutoFewShots.Generate)`, `ForeachLoopGraphNode(Tasks)`, `ResultAggregator`, `PIIFilter`, `RetryPolicyGraphNode`.

## Regras/Políticas e Guardrails
- Remover/mascarar PII; respeito a sensibilidade.
- Garantir citações nos exemplos que exigem evidência.
- Versionamento sem impacto imediato: `rollout` controlado com fallback.

## Saídas e Artefatos
- `few_shots` armazenados em `prompt_profiles` por projeto/idioma/tarefa, com `version`, `created_at`, `metrics`.
- Relatórios de curadoria: contagens por classe, taxa de dedup, taxa de PII removida.

## Parâmetros e Ajustes
- `target_per_class`, `max_examples`, `version_label`, `rollout_percent`.
- Classificadores de qualidade e thresholds por tarefa.

## Métricas e Critérios de Aceite
- Métricas: melhoria de relevância/groundedness/cobertura, queda de latência/custo, taxa de rejeição por PII.
- Aceite: melhora estatisticamente significativa em A/B; exemplos auditáveis e citáveis; rollback seguro disponível.

## Rastreabilidade
- Roadmap: seções 12 e 16 em `docs/roadmap.md`.
- Roadmap técnico: seções 1.3, 7, 11 e 12 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Contaminação por exemplos enviesados → balanceamento e revisão.
- Vazamento de PII → filtros obrigatórios e auditoria.
- Drift de domínio → versões por período e re-avaliação periódica.

## Exemplos
- Gerar few-shots para answering (pt):
```
POST /v1/projects/{id}/prompts/few-shots/generate
{
  "task": "answering",
  "filters": { "lang": "pt", "tema": "Conformidade" },
  "params": { "target_per_class": 12, "dedup": true, "pii_mask": true, "version_label": "v2025-09" }
}
```

## Variantes e Extensões
- Active learning: priorizar candidatos por incerteza/impacto.
- Fine-tuning leve (LoRA) usando exemplos aprovados (fora do escopo imediato).
