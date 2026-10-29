ETAPA: design · change [nome-da-change]

Proposal aprovado:
[PROPOSAL.MD]

Specs delta aprovados:
[SPECS DELTA]

Instruções do OpenSpec (saída de `openspec instructions design --change [nome-da-change] --json`):
[INSTRUÇÕES DO DESIGN]

Árvore de diretórios atual:
[ÁRVORE]

Contratos de dados atuais que esta change toca:
[config_schema.json / annotation_output.json, ou "nenhum"]

Tarefa: escrever o design.md desta change.
- Use as seções do `template`.
- Explique como cada requirement será atendido respeitando as camadas:
  UI -> Services -> Domain e Repositories; o Storage implementa os Repositories e
  é o único que acessa SQLite e arquivos (ZIP/JSON). O Domain não depende de
  nenhuma outra camada.
- Para cada decisão: decisão, alternativas consideradas, motivo e consequência.
  Formatos de arquivo (pacote, exportação, anotações) são sempre decisões
  explícitas com alternativas.
- Diga o que entra em cada camada em nível de responsabilidade. Você pode propor
  nomes de módulos e classes (em inglês, PEP 8) e assinaturas ilustrativas, mas
  não escreva implementação.
- Marque com [2º DEV] toda decisão que envolva nova dependência, mudança de schema
  do banco, contrato de dados ou assinatura pública usada por outras changes.
- Nada de rede, nuvem, CDN ou telemetria. Nada de perguntas ou classes fixas no código.
- Se a change não exige decisões novas, mantenha o design curto e escreva
  "segue os padrões existentes".
- Não altere o comportamento das specs. Se elas não bastarem para decidir, registre
  em "Open Questions" em vez de completar por conta própria.

Entregue: o caminho em uma linha, o arquivo completo em um bloco markdown e,
depois, a lista das decisões marcadas [2º DEV].
