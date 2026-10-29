ETAPA: tasks · change [nome-da-change]

Proposal, specs delta e design aprovados:
[PROPOSAL.MD]
[SPECS DELTA]
[DESIGN.MD]

Instruções do OpenSpec (saída de `openspec instructions tasks --change [nome-da-change] --json`):
[INSTRUÇÕES DO TASKS]

Árvore de diretórios atual:
[ÁRVORE]

Tarefa: escrever o tasks.md desta change.

Formato de cada task (o Fluxo de Implementação lê estes campos):
## 1. <Grupo>
- [ ] 1.1 <Título no imperativo> (Cenários: <Requirement> / <Scenario>; Arquivos: <caminhos a criar ou alterar>; Teste: <arquivo de teste> - `pytest <caminho> -q`)

Regras:
- Cada task cabe em uma conversa de implementação (~1h) e em um PR pequeno.
- A ordem respeita as dependências do design: Domain -> Repositories -> Storage
  -> Services -> UI.
- O teste do comportamento fica na mesma task que o implementa, nunca em um grupo
  final. Task sem cenário (estrutura inicial, por exemplo) usa
  "Cenários: nenhum (infraestrutura)" e diz como verificar.
- Caminhos seguem a árvore enviada; arquivos novos seguem a estrutura existente.
  Se ela ainda não existir, use a do design e registre isso em "Questões em aberto".
- Não crie tasks que editem proposal, specs, design ou tasks.
- O último grupo é "## N. Verificação": suíte completa, checagens de arquitetura
  (grep de termos proibidos por camada) e aceitação manual dos cenários
  "(verificação manual)". Esse grupo não cria testes novos.
- No fim do arquivo, a seção "## Cobertura" com a tabela
  | Requirement | Scenario | Tasks |, listando TODOS os cenários das specs.
  Nenhum cenário pode ficar sem task.

Entregue: o caminho em uma linha, o arquivo completo em um bloco markdown e,
depois, "Questões em aberto".
