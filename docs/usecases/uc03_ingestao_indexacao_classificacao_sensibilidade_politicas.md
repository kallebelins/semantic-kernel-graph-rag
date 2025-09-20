# Usecase: Classificação de sensibilidade e políticas de privacidade

## Contexto e Problema
Necessário rotular trechos como Público/Interno/Confidencial e aplicar políticas de exibição e indexação.

## Objetivo
Classificar sensibilidade por `TextUnit` e aplicar redação/mascaramento conforme perfil.

## Atores e Escopo
- Serviços: `PIIFilter`, `Guardrails`
- Escopo: Classificação + aplicação de políticas

## Pré-condições
- `TextUnit` disponíveis
- Políticas por projeto definidas

## Entradas
- `TextUnit[]`
- Perfis de usuário/política

## Fluxo (alto nível)
- Classifier → label de sensibilidade
- Aplicar políticas (redação/mascaramento)
- Atualizar índices (excluir/filtrar conforme label)

## Regras/Políticas e Guardrails
- Citações obrigatórias preservam metadados mesmo com redação
- Auditoria de decisões por unidade

## Saídas e Artefatos
- Labels de sensibilidade por unidade
- Logs/auditoria de políticas aplicadas

## Parâmetros e Ajustes
- Limiar de confiança de classificação
- Campos sujeitos à redação

## Métricas e Critérios de Aceite
- Precisão da classificação por amostras auditadas
- Ausência de vazamento de PII em respostas públicas

## Rastreabilidade
- Roadmap: 9) Segurança, Privacidade e Governança
- Roadmap técnico: 8) Segurança, LGPD e Governança

## Riscos e Considerações
- Falsos negativos de PII
- Políticas conflitantes

## Exemplos
- Redação de CPF/Emails em unidades públicas

## Variantes e Extensões
- Re-treinamento de classificadores por setor/idioma
