ETAPA: specs · change [nome-da-change]

Proposal aprovado:
[PROPOSAL.MD]

Critérios de aceitação e decisões aprovados no Explore:
[RESUMO DO EXPLORE]

Instruções do OpenSpec (saída de `openspec instructions specs --change [nome-da-change] --json`):
[INSTRUÇÕES DO SPECS]

Specs principais atuais das capacidades "Modified":
[SPECS PRINCIPAIS, ou "nenhuma"]

Tarefa: escrever um spec delta para cada capacidade listada no proposal.

Formato obrigatório (o `openspec validate --strict` depende dele):
- Seções `## ADDED Requirements`, `## MODIFIED Requirements` e
  `## REMOVED Requirements`, só as que houver.
- `### Requirement: <nome> (USxx)`. O texto do requirement contém MUST ou SHALL
  (ex.: "O sistema MUST permitir ...").
- Cada requirement tem ao menos um `#### Scenario: <nome>` (exatamente 4 #), com
  linhas `- **WHEN** ...`, `- **THEN** ...` e, se preciso, `- **AND** ...`.
- MODIFIED: copie o requirement inteiro da spec principal e altere. O que você
  omitir será apagado no arquivamento.

Conteúdo:
- Todo CA aprovado vira ao menos um Scenario.
- Inclua cenários de erro e validação (entrada inválida, item inexistente,
  cancelamento), não só o caminho feliz.
- Descreva comportamento observável. Não mencione tecnologia, framework, classe,
  tabela ou arquivo de código.
- Perguntas, respostas e classes são configuradas pelo coordenador: não trate
  exemplos de domínio como regra fixa.
- Cenários que só podem ser conferidos pela interface terminam o nome com
  "(verificação manual)".
- Não crie requirement que não venha de um CA ou de uma decisão do resumo. Se um CA
  não puder virar cenário sem decisão nova, pare e registre em "Questões em aberto".

Entregue: para cada arquivo, o caminho em uma linha e o conteúdo completo em um
bloco markdown; depois, a tabela "Cobertura CA" (| CA | Requirement | Scenario |);
por fim, "Questões em aberto".
