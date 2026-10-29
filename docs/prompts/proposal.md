ETAPA: proposal · change [nome-da-change]

Resumo do Explore aprovado:
[RESUMO DO EXPLORE]

Instruções do OpenSpec (saída de `openspec instructions proposal --change [nome-da-change] --json`):
[INSTRUÇÕES DO PROPOSAL]

Capacidades que já existem em openspec/specs/:
[LISTA: saída de `ls openspec/specs`, ou "nenhuma"]

Tarefa: escrever o proposal.md desta change.
- Use as seções do `template`, na mesma ordem.
- Why: o problema e o valor, citando os IDs das histórias cobertas (ex.: US01, US02).
- What Changes: uma mudança de comportamento observável por item. Inclua o item
  "Fora de escopo", indicando a change do plano onde cada ponto entra.
- Capabilities: só nomes do mapa. "New" se não estiver na lista de existentes,
  "Modified" se estiver. Se nenhum nome servir, diga isso explicitamente e proponha
  o novo nome como questão em aberto.
- Impact: camadas e contratos de dados afetados (config_schema.json,
  annotation_output.json), sem nomes de classes, tabelas ou arquivos de código.
- Tudo vem do Resumo do Explore. Não acrescente comportamento, critério ou decisão,
  e não inclua detalhes de implementação.
- Mantenha as questões em aberto do resumo em uma seção final "Questões em aberto".

Entregue: o caminho (`resolvedOutputPath`) em uma linha; o arquivo completo em um
único bloco markdown; depois do bloco, "Pontos para validação" com o que eu devo
conferir.
