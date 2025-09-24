# Integração Social/Organizacional (Pessoas, Equipes, Projetos) e Relações com Conteúdos

## Contexto e Problema
Perguntas organizacionais exigem contexto sobre pessoas, equipes e projetos: quem participou, aprovou, discutiu; quais conteúdos citam pessoas; qual o envolvimento por departamento. É necessário integrar uma camada social/organizacional ao grafo, mantendo governança e privacidade.

## Objetivo
Modelar e popular a camada social/organizacional com nós e arestas dedicadas, relacionando-a aos conteúdos (TextUnits) e às entidades/claims, habilitando consultas e insights por envolvimento, responsabilidade e impacto.

## Atores e Escopo
- Usuários: RH/PMO/Compliance, gestores, analistas.
- Serviços: `GraphStore`, `IEExtractor` (menções de pessoas/organizações), `PIIFilter`, `Router/Query`.
- Escopo cobre: criação de nós `Person`, `Team`, `ProjectOrg` e arestas; consultas e filtros combinados.
- Fora do escopo: HRIS/ERP conectores (podem ser futuras integrações de enriquecimento).

## Pré-condições
- Esquema social habilitado no projeto.
- Políticas de privacidade e PII definidas (mascaramento sob perfis).
- Ingestão concluída e IE com detecção de menções de pessoas/organizações.

## Entradas
- Fontes de nomes/equipes/projetos (lista corporativa) e menções extraídas dos conteúdos.
- Parâmetros: `match_rules` (alias/normalização), `confidence_threshold`, `mask_pii`.
- Filtros de consulta: `team`, `dept`, `project`, `role`, `lang`, `sensitivity`.

## Fluxo (alto nível)
1) Normalizar e resolver menções de pessoas/orgs (aliases, lematização, regras de matching).
2) Criar/atualizar nós sociais e relacionamentos:
   - Nós: `(:Person {id, name, role, dept})`, `(:Team {id, name})`, `(:ProjectOrg {id, name})`.
   - Arestas: `(:Person)-[:PARTICIPATED_IN]->(:ProjectOrg)`, `(:Person)-[:MEMBER_OF]->(:Team)`, `(:Person)-[:MENTIONED_IN]->(:TextUnit)`, `(:Person)-[:REL {type}]->(:Entity)`.
3) Conectar a entidades/claims e a `TextUnit` com citações (page/time_range/bbox quando multimodal).
4) Consultas suportadas: envolvimento por projeto, responsáveis por tema, pessoas citadas em procedimentos, impacto por equipe.
5) Aplicar `PIIFilter` antes de retornar resultados.

- Nós SKGraph (exemplos): `GraphMergeNode` estendido para camada social, `IEExtractorNode` com tipos sociais, `LocalGraph`/`GlobalQFS` com filtros sociais.

## Regras/Políticas e Guardrails
- RBAC/ABAC por perfil (acesso a dados pessoais mínimo necessário).
- PII: mascarar nomes/atributos para perfis externos; expor apenas agregados quando necessário.
- Consentimento e direito ao esquecimento por `PersonId`.

## Saídas e Artefatos
- Grafo enriquecido com camada social/organizacional.
- Respostas de consulta com `citations[]` e metadados sociais (quando permitido).
- Relatórios por equipe/projeto (opcional) e KPIs de envolvimento.

## Parâmetros e Ajustes
- `confidence_threshold` para vincular menções a pessoas.
- `match_rules` para aliases; `mask_pii` por perfil; `max_hop` social.

## Métricas e Critérios de Aceite
- Métricas: precisão/recall de matching de pessoas, latência adicional, incidentes de PII.
- Aceite: consultas sociais funcionam; políticas de PII respeitadas; citações presentes quando exibindo conteúdos.

## Rastreabilidade
- Roadmap: seção 16 (Integração com grafos de pessoas/processos) em `docs/roadmap.md`.
- Roadmap técnico: seção 3.2.1 e 16 em `docs/roadmap_tecnico.md`.

## Riscos e Considerações
- Ambiguidade de nomes (homônimos) → exigir contexto (departamento/projeto) e limiar de confiança.
- Privacidade: aplicar minimização e mascaramento; auditoria de acessos.

## Exemplos
- Consulta: “Quem participou do Projeto X e é citado em procedimentos de segurança?”
```
POST /v1/query
{
  "project_id": "proj-123",
  "question": "Quem participou do Projeto X e é citado em procedimentos de segurança?",
  "mode": "local_graph",
  "filters": { "project": "Projeto X", "tema": "Segurança" },
  "params": { "depth": 2, "beam_width": 4 }
}
```

## Variantes e Extensões
- Enriquecimento via HRIS/AD (organogramas, lotações) e integração com calendários/PMO.
- Insights sociais: centralidade por equipe, gargalos, responsabilidade por conformidade.
