# RBAC/ABAC, redação/mascaramento de PII, direito ao esquecimento

## Contexto e Problema
Ambientes corporativos exigem controle de acesso granular (RBAC/ABAC), proteção de PII e mecanismos de remoção seletiva (direito ao esquecimento). Em RAG híbrido, filtros e políticas devem ser aplicados desde a montagem de contexto até a resposta, preservando citações e auditabilidade.

## Objetivo
Aplicar políticas de acesso e privacidade por `tenant/project/tag/setor/tema/sensibilidade`, com redação/mascaramento de PII em prompts/contexts/respostas conforme perfil do usuário. Implementar direito ao esquecimento por `docId/unitId/entityId`, propagando remoções para índices vetoriais, BM25 e grafo.

## Atores e Escopo
- Usuário final (Scopes: `read:rags`), Administrador (Scopes: `admin:projects`), Serviços internos
- Serviços: Auth (Keycloak/OAuth2), `PIIFilter`, `Guardrails`, índices (`IVectorIndex`, `ILexicalIndex`), `IGraphStore`
- Dentro: avaliação de políticas no roteador e nós de consulta, redação de PII, exclusões seletivas
- Fora: gestão de identidade do IdP (configuração externa)

## Pré-condições
- Classificação de sensibilidade por `TextUnit` e metadados `{tags,setor,tema,lang}` populados
- Políticas definidas por projeto: `{rbac_roles, abac_rules, pii_redact, retention_ttl, legal_holds[]}`
- Integração com IdP (Keycloak) e propagação de `userId/roles/attrs` para o `GraphContext`

## Entradas
- Consulta: `{project_id, question, filters, params, userContext{userId, roles[], attrs{dept, region, clearance}}}`
- Políticas ativas do projeto e taxonomias
- Requisição de esquecimento: `{scope: doc|unit|entity, ids[], reason}`

## Fluxo (alto nível)
- Etapa 1: Avaliar RBAC/ABAC antes de qualquer recuperação (pré-filtro em índices e grafo)
- Etapa 2: Aplicar `PIIFilter` na montagem de contexto (prompts/trechos) com marcação de redações
- Etapa 3: Gerar resposta com citações obrigatórias; aplicar máscara na resposta final quando necessário
- Etapa 4: Registrar auditoria (`who, what, when, policies_applied`)
- Etapa 5 (Esquecimento): resolver escopo, excluir/atualizar entradas em `IVectorIndex`, `BM25`, `IGraphStore` e artefatos derivados; invalidar caches
- Nós SKGraph: `Guardrails`/`PIIFilter`/`PolicyEvaluator` acoplados ao roteador e aos nós de resposta

## Regras/Políticas e Guardrails
- Citações obrigatórias em respostas decisórias; bloquear se origem for restrita ao perfil
- ABAC por `{tags,setor,tema,lang,sensitivity}` e atributos do usuário `{dept, region, clearance}`
- Redação de PII configurável: `{cpf, rg, email, phone, address}` com mascaramento consistente
- Retenção/TTL e "legal hold" por projeto; auditoria imutável

## Saídas e Artefatos
- Resposta com citações `doc_id/unit_id` (ou `community_id` quando global) já com máscaras aplicadas
- `auditLog` por consulta: `{userId, projectId, mode, filters, policies_applied[], redactions[], denied_reasons?}`
- Registros de esquecimento: `{scope, ids[], status, ts}` e evidências de propagação

## Parâmetros e Ajustes
- `pii_redact.enabled`, `pii_redact.fields[]`, `pii_redact.strategy {mask|remove|hash}`
- `allowed_sensitivity {Public|Internal|Confidential}` por papel
- `filters`: `{tags[], setor, tema, lang}` influenciando ABAC
- `erasure.concurrent_ops`, `erasure.retry_policy`

## Métricas e Critérios de Aceite
- Zero vazamentos conhecidos de PII em respostas quando `pii_redact.enabled=true`
- Consultas negadas ou parcialmente atendidas corretamente conforme políticas (testes RBAC/ABAC)
- SLA de esquecimento: 95% das requisições aplicadas e refletidas em ≤ 10 min
- Traços de auditoria presentes para 100% das consultas

## Rastreabilidade
- Roadmap: seções 9 (Segurança/Privacidade/Governança)
- Roadmap técnico: seções 4 (Citações/Metadados), 6 (Endpoints), 8 (Segurança), 13 (Parâmetros)

## Riscos e Considerações
- Redação excessiva reduz qualidade de resposta: calibrar campos e estratégias por intent
- Inconsistências entre índices/grafo após esquecimento: usar outbox e reconciliação periódica
- Penalidade de latência por filtros: aplicar estratégias de pré-filtragem facetada e caching seguro

## Exemplos
- Política RBAC/ABAC por projeto (trecho):
```json
{
  "roles": {"analyst": {"allowed_sensitivity": ["Public", "Internal"]}},
  "abac": {"rules": [{"attr": "dept", "equals": "Compliance", "allow": ["Confidential"]}]},
  "pii_redact": {"enabled": true, "fields": ["email", "phone"], "strategy": "mask"}
}
```

- Resposta redigida (exemplo de máscara):
"O contato do DPO é <email_redigido>@empresa.com e o telefone é (***)***-1234."

- Esquecimento (payload):
```json
{ "scope": "unit", "ids": ["u-789"], "reason": "GDPR request" }
```

## Variantes e Extensões
- Multimodal: políticas específicas por mídia (áudio: `time_range`, imagem: `bbox`); filtros por sensibilidade por mídia
- Pseudonimização e data minimization orientadas por intent
