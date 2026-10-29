# Fluxo de Verificação e Finalização

SDD com OpenSpec CLI · harness manual (agentless) · Tarefa 1 · Documento 2 de 2

## Premissa e papéis

O chat/LLM apenas gera texto. O desenvolvedor assume o papel do *harness*, executando manualmente o que um agente faria:

| Papel do harness | Quem faz neste fluxo |
| --- | --- |
| Gerenciar contexto | O dev monta o "pacote de contexto" a cada task, seguindo o `docs/guia_de_contexto.md` |
| Loop de execução | O dev cola o prompt, lê a resposta, aplica, testa e devolve o resultado |
| Ferramentas (read/edit/terminal/git) | O dev executa. O LLM só pede, no formato `PRECISO DE: ...` |
| Permissões | O dev (e um 2º dev, quando exigido) aprova antes de aplicar |

Os comandos `/opsx:*` (explore, propose, apply, verify, archive) pressupõem um agente. Aqui eles são substituídos por comandos da CLI `openspec` e por prompts manuais. Se nenhum LLM for usado em alguma etapa, "enviar o prompt" passa a ser "implementar manualmente"; o restante do fluxo não muda.

## Placeholders usados neste documento

| Placeholder | Significado |
| --- | --- |
| `[nome-da-change]` | Nome da change OpenSpec (kebab-case, em inglês) |
| `[N.M]` | Número da task no `tasks.md` |
| `[US-xx]` | História de usuário que originou a change |
| `[artefato]` | proposal, specs, design ou tasks |
| `[arquivo]`, `[diretorio-da-camada]`, `[termo-proibido]` | Caminhos e termos específicos do projeto |

## Visão geral

V1 Testes → V2 Loop de correção → V3 Critérios da spec → V4 Aceitação manual → V5 Concluir task → V6 Review/PR → V7 Condições de arquivamento → V8 Arquivar → V9 Retro.

Parte do ponto em que o documento de Implementação termina (Passo 7).

## V1 – Rodar os testes (o dev faz Run Tests)

- (a) testes da task; (b) suíte completa com `pytest`; (c) linter/formatter, se o repositório tiver.
- Guarde a saída completa dos comandos.

## V2 – Se falhar, devolver o erro ao LLM

Na mesma conversa da task (o `project.md` não precisa ser reenviado):

```text
Os testes falharam após a task [N.M].
Comando: [pytest ...]
Alterações realizadas: [arquivos alterados nesta task]
Saída (teste que falhou + traceback): [colar]
Cenário da spec relacionado: [nome do cenário + WHEN/THEN]
Código atual dos arquivos envolvidos: [arquivos como estão no disco agora]
Regras: corrija a causa, não ajuste o teste só para passar. Se o teste
estiver errado, justifique com base na spec. Mude o mínimo, respeite as
regras do project.md e liste os arquivos alterados.
```

- Sempre envie o arquivo como está no disco, não a versão que o LLM lembra.
- Em logs longos, envie o teste que falhou e o *traceback*, não a saída inteira.
- Máximo de 3 iterações. Depois disso, o dev depura sozinho ou investiga se a spec está ambígua.
- Se for a spec, volte ao Fluxo de Especificação: `openspec instructions [artefato] --change [nome-da-change] --json`, corrija pelo laço de correção e rode `openspec validate [nome-da-change] --strict`.
- Ordem de autoridade: spec > teste > código.
- Depois de cada correção, volte ao V1.

## V3 – Verificar os critérios da spec (o `/opsx:verify` manual)

Monte a matriz em `docs/sdd/runs/[nome-da-change].md`, uma linha por cenário da spec. Parta da tabela "Cobertura" do `tasks.md`, que já lista todos os cenários e as tasks de cada um.

| Requirement / Cenário | Task | Teste automatizado | Verificação manual | Status |
| --- | --- | --- | --- | --- |
| [Requirement 1] / [Cenário: caso de sucesso] | [N.M] | [nome_do_teste] | [o que foi conferido na interface] | OK |
| [Requirement 1] / [Cenário: validação ou erro] | [N.M] | [nome_do_teste] | [o que foi conferido] | Pendente |
| [Requirement 2] / [Cenário: cancelamento] | [N.M] | [nome_do_teste] | [o que foi conferido] | Falhou |

- **Completude:** todo cenário tem evidência e todas as tasks estão `[x]`. Cenários marcados como "(verificação manual)" na spec têm evidência no V4.
- **Correção:** o comportamento bate com a spec.
- **Coerência:** o código segue o `design.md` e as regras invioláveis do `project.md`. Exemplo de checagem mecânica: `grep -rn "[termo-proibido]" [diretorio-da-camada]` deve voltar vazio (ex.: `sqlite3` fora da camada Storage, ou `PySide6` no Domain).

Opcional: peça a um LLM, em chat novo, uma revisão com a spec, o design, `git diff develop...HEAD` e a saída dos testes:

```text
Você é revisor de uma change OpenSpec. Não escreva código novo.
[CONTEXTO GLOBAL: docs/project.md]
[SPEC DA CHANGE]
[DESIGN.MD]
[DIFF: git diff develop...HEAD]
[SAÍDA DOS TESTES]
Responda em tabela:
1. Requisitos/cenários sem evidência de implementação ou de teste.
2. Trechos do diff que violam o design.md ou as regras do contexto global.
3. Comportamentos implementados que a spec não pede (escopo extra).
Se não houver itens em uma categoria, escreva "nenhum".
```

O veredito do LLM é só pista: o dev confirma cada item lendo o código ou rodando o app.

## V4 – Teste manual de aceitação

- Rode o app e percorra o roteiro da história [US-xx] e dos cenários da spec: fluxo principal, validações e mensagens de erro, cancelamento/volta e a transição para a próxima etapa.
- Registre o resultado e a evidência (observação ou captura de tela) na matriz do V3.

## V5 – Quando marcar a task como concluída (Definition of Done)

- [ ] Testes da task e suíte completa passando.
- [ ] Cada cenário coberto pela task tem evidência na matriz.
- [ ] O `git diff` contém só arquivos do escopo.
- [ ] Regras de arquitetura verificadas.
- [ ] Commit feito com referência à task: `feat([nome-da-change]): task [N.M] - [título]`.

Cumprido tudo, o dev troca `- [ ]` por `- [x]` no `tasks.md` (nenhum agente faz isso) no mesmo commit e confere o progresso com `openspec list`. Esta é a única edição do `tasks.md` permitida fora do Fluxo de Especificação. Depois, volte ao Passo 1 do documento de Implementação com contexto novo.

## V6 – Review e PR

- Abra o PR `feature/[nome-da-change]` → `develop`, por task ou ao fim das tasks, conforme o tamanho da change.
- O revisor deve ser outra pessoa e usa a matriz do V3 como checklist.

## V7 – Quando a change pode ser arquivada

- Todas as tasks estão `[x]`, incluindo as do grupo "Verificação" do `tasks.md`.
- `openspec validate [nome-da-change] --strict` passa.
- A matriz não tem pendências. Divergências só são aceitas se documentadas: ou você ajusta o código, ou altera a spec pelo Fluxo de Especificação. Nunca arquive uma spec que não corresponde ao código.
- O PR está aprovado.

## V8 – Arquivar

- Rode `openspec archive [nome-da-change]`: a change vai para o arquivo e os deltas são aplicados às specs principais. Não use o `mv` manual descrito na skill `opsx-archive`: sem agente, ele arquiva sem sincronizar as specs.
- Rode `openspec validate --all --strict` para garantir que as specs principais continuam válidas.
- Revise o diff de `openspec/specs/`: requisitos `MODIFIED` substituem o requisito inteiro, então confira que nenhum cenário antigo sumiu sem intenção. Confirme com `openspec list` que a change saiu da lista ativa.
- Faça um commit separado, `chore(openspec): archive [nome-da-change]`, e dê merge do PR em `develop`.

## V9 – Retro rápida

- Ajuste os prompts em `docs/sdd/prompts/` conforme o que travou durante a change. Isso alimenta a avaliação do fluxo na próxima reunião.

## Alterações de alinhamento (28/10/2026)

Ajustes feitos na conversão do PDF para que este fluxo converse com os demais:

1. **Validação estrita:** `openspec validate` passou a usar `--strict` (V2, V7, V8), igual ao Fluxo de Especificação.
2. **Prompt de correção (V2):** inclui "Alterações realizadas" e a regra do log curto, que vêm da seção 8 do guia de contexto. Este passa a ser o modelo único de correção de testes.
3. **Matriz (V3):** ganhou a coluna "Task" e parte da tabela "Cobertura" que o Fluxo de Especificação exige no `tasks.md`.
4. **Coerência (V3):** a checagem usa as regras invioláveis do `project.md`, com exemplos concretos de termos proibidos por camada.
5. **V5:** explicita que marcar `- [x]` é a única edição de `tasks.md` fora da especificação, coerente com o Passo 5 da Implementação.
6. **V8:** alerta contra o `mv` da skill e lembra que `MODIFIED` substitui o requisito inteiro.
