### Contribuindo para o Semantic Kernel Graph RAG

Obrigado por querer contribuir! Este projeto foca em RAG híbrido de nível corporativo, orquestrado como grafos com SKGraph. Para manter qualidade e alinhamento, siga as orientações abaixo.

## Valores e princípios
- **Qualidade corporativa**: confiabilidade, auditabilidade, segurança e custo previsível.
- **Arquitetura guiada por grafos**: pipelines como grafos (SKGraph) e decisões documentadas.
- **Transparência**: PRs pequenos e bem descritos; decisões registradas em docs.

## Código de Conduta
Adotamos um ambiente inclusivo e respeitoso. Ao contribuir, você concorda em agir com cordialidade e profissionalismo.

## Como começar
1. Faça fork do repositório e crie uma branch a partir de `main`.
2. Nomeie a branch: `feat/…`, `fix/…`, `docs/…`, `chore/…`, `refactor/…`.
3. Desenvolva com mudanças focadas e atômicas.
4. Abra um Pull Request (PR) descrevendo claramente o problema, a abordagem e o impacto.

## Alinhamento com o roadmap e tarefas
- O desenvolvimento deve seguir `docs/roadmap.md` e `docs/roadmap_tecnico.md`.
- As tarefas são organizadas em `docs/tasks.md`. Se precisar de subtarefas, adicione-as primeiro.
- Ao concluir algo que conste em `docs/tasks.md`, atualize o status de `[ ]` para `[x]`.

## Ambiente de desenvolvimento
- Requisitos mínimos: .NET 8 SDK e Git.
- Dependências específicas de módulos (ex.: Neo4j, ferramentas de embeddings) serão indicadas na documentação dos respectivos diretórios.
- Para imagens e materiais de comunicação, use `assets/`.

## Padrões de código e referências
- Para código C#/.NET relacionado a grafos (criação, consulta, travessia ou modificação):
  - Siga estritamente os padrões e exemplos do `SemanticKernel.Graph`.
  - Referências oficiais:
    - NuGet: [SemanticKernel.Graph](https://www.nuget.org/packages/SemanticKernel.Graph)
    - Documentação: [skgraph.dev](https://skgraph.dev/)
    - Exemplos: [semantic-kernel-graph-docs/examples](https://github.com/kallebelins/semantic-kernel-graph-docs/tree/main/examples)
- Princípios gerais de código:
  - Nomeação descritiva, funções coesas e tratamento explícito de erros.
  - Evite comentários triviais; documente decisões e invariantes.
  - Prefira early-return e evite nesting profundo.
  - Mantenha formatação consistente e não misture estilos de indentação.

## Commits e mensagens
- Use Conventional Commits: `feat: …`, `fix: …`, `docs: …`, `chore: …`, `refactor: …`, `test: …`.
- Escreva mensagem curta no título e detalhe no corpo quando necessário.
- Relacione issues: `Closes #123` quando aplicável.

## Pull Requests (PRs)
Inclua no PR:
- Contexto do problema e objetivo da mudança.
- Abordagem técnica e trade-offs.
- Impactos em performance, segurança, custos e observabilidade (quando relevante).
- Testes e/ou evidências de funcionamento (logs/prints/trace) quando aplicável.
- Atualizações de documentação (ex.: `docs/usecases/`, `docs/usegraphs/`, `docs/roadmap*`).

Checklist sugerido:
- [ ] Build local passou e mudanças são atômicas.
- [ ] Documentação atualizada quando necessário.
- [ ] Sem segredos/credenciais no código ou histórico.
- [ ] Aderência aos padrões do `SemanticKernel.Graph` quando tocar grafo.

## Issues
Ao abrir uma issue, inclua:
- Objetivo, motivação e escopo.
- Cenário de uso (link para `docs/usecases/` e/ou `docs/usegraphs/` se existir).
- Passos de reprodução (se bug) e ambiente.
- Alternativas consideradas (opcional) e sugestões de abordagem.

## Documentação
- Mantenha o conteúdo organizado em `docs/` conforme a estrutura existente:
  - Roadmaps: `docs/roadmap.md`, `docs/roadmap_tecnico.md`
  - Usecases: `docs/usecases/`
  - Usegraphs: `docs/usegraphs/`
- Materiais visuais e banners: `assets/`.

## Licença
- Este projeto é licenciado sob Apache-2.0 com Commons Clause (no-sale).
- Ao contribuir, você concorda que suas contribuições serão licenciadas sob os mesmos termos.

## Onde aprender mais
- NuGet: `https://www.nuget.org/packages/SemanticKernel.Graph`
- Documentação SKGraph: `https://skgraph.dev/`
- Exemplos: `https://github.com/kallebelins/semantic-kernel-graph-docs/tree/main/examples`

Obrigado por colaborar! Sua participação acelera a construção de um RAG híbrido confiável — base para um novo Copilot corporativo.


