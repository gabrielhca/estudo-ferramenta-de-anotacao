# Fluxo de Especificação

SDD com OpenSpec CLI · harness manual (agentless) · Tarefa 1 · Documento 0 (antecede Implementação e Verificação)

## Premissa e papéis

O chat/LLM apenas gera texto. O desenvolvedor assume o papel do *harness*, executando manualmente o que um agente faria:

| Papel do harness | Quem faz neste fluxo |
| --- | --- |
| Gerenciar contexto | O dev monta o pacote de cada etapa (tabela "Pacote de contexto por etapa", abaixo), seguindo o `docs/guia_de_contexto.md` |
| Loop de execução | O dev cola o prompt, lê a resposta, salva o artefato, valida e commita |
| Ferramentas (CLI/read/git) | O dev roda `openspec` e cola a saída. O LLM só pede, no formato `PRECISO DE: ...` |
| Decisões de negócio | Coordenador/equipe. O LLM apresenta opções; nunca escolhe |
| Permissões | O dev (e um 2º dev, quando exigido) aprova antes de commitar |

Os comandos `/opsx:explore` e `/opsx:propose` pressupõem um agente que roda a CLI e grava arquivos. Aqui:

- `/opsx:explore` vira o prompt **explore** (etapa E1).
- `/opsx:propose` gera todos os artefatos de uma vez. Neste fluxo ele é **quebrado em quatro etapas** (E2 a E5), uma por artefato, para que cada uma tenha validação humana antes da próxima.
- `opsx-sync` e `opsx-archive` **não vão para o LLM**: a sincronização e o arquivamento são feitos por `openspec archive` (V8 da Verificação).

## Placeholders usados neste documento

| Placeholder | Significado |
| --- | --- |
| `[nome-da-change]` | Nome da change do plano no `docs/mapa-de-capacidades.md` (kebab-case) |
| `[US-xx]` | Histórias de usuário da change, copiadas pelo ID de `docs/historias-de-usuario.md` |
| `[capacidade]` | Nome de capacidade do mapa (ex.: `projeto-anotacao`) |
| `[artefato]` | `proposal`, `specs`, `design` ou `tasks` (IDs do schema `spec-driven`) |
| `Dxx`, `CAxx` | Decisão de negócio e critério de aceitação numerados no Explore |

## Visão geral

E0 Preparação → E1 Explore → E2 Proposal → E3 Specs → E4 Design → E5 Tasks → E6 Fechamento.

Laço de correção: a qualquer momento (inclusive na Implementação/Verificação), um problema em um artefato volta para a etapa que o produziu.

Depois do E6, segue o documento de Implementação (Passo 0).

## Qual skill acompanha cada etapa

Os arquivos `SKILL-*.md` e `opsx-*.md` têm praticamente o mesmo conteúdo. **Envie só um deles por chat** (o `SKILL-*`), para não gastar contexto em duplicata.

| Etapa | Skill enviada | Comando CLI que o dev roda | Prompt | Artefato produzido |
| --- | --- | --- | --- | --- |
| E1 Explore | `SKILL-explore.md` | `openspec list --json` (opcional) | `explore.md` + `explore-fechar.md` | Resumo do Explore em `docs/sdd/runs/[nome-da-change].md` |
| E2 Proposal | `SKILL-propose.md` | `openspec instructions proposal --change [nome-da-change] --json` | `proposal.md` | `openspec/changes/[nome-da-change]/proposal.md` |
| E3 Specs | `SKILL-propose.md` | `openspec instructions specs --change [nome-da-change] --json` | `specs.md` | `openspec/changes/[nome-da-change]/specs/[capacidade]/spec.md` |
| E4 Design | `SKILL-propose.md` | `openspec instructions design --change [nome-da-change] --json` | `design.md` | `openspec/changes/[nome-da-change]/design.md` |
| E5 Tasks | `SKILL-propose.md` | `openspec instructions tasks --change [nome-da-change] --json` | `tasks.md` | `openspec/changes/[nome-da-change]/tasks.md` |
| Correção | a da etapa corrigida | `openspec instructions [artefato] --change [nome-da-change] --json` | `correcao-artefato.md` | Artefato corrigido |

Os prompts ficam em `docs/sdd/prompts/`. Cada bloco `text` deste documento é o conteúdo do arquivo de mesmo nome.

## Pacote de contexto por etapa

Regra geral: **um chat novo por etapa**, colando o `base.md` + o pacote da etapa. Nunca envie código, imagens, datasets, `.db` ou `.zip`.

| Etapa | Global | Da mudança | De código | Observação |
| --- | :---: | --- | :---: | --- |
| E1 Explore | `project.md` | Histórias da change, mapa de capacidades, specs principais das capacidades envolvidas (se existirem) | — | Specs principais evitam contradizer o que já foi arquivado |
| E2 Proposal | `project.md` | Resumo do Explore, instruções do proposal, lista de `openspec/specs/` | — | A lista decide "New" vs "Modified" |
| E3 Specs | `project.md` | `proposal.md`, Resumo do Explore (critérios aprovados), instruções do specs, specs principais das capacidades "Modified" | — | Sem os critérios, a regra "todo CA vira Scenario" não pode ser cumprida |
| E4 Design | `project.md` | `proposal.md`, specs delta, instruções do design, contratos de dados tocados pela change | Árvore de diretórios | Árvore gerada na hora (seção 5 do guia) |
| E5 Tasks | `project.md` | `proposal.md`, specs delta, `design.md`, instruções do tasks | Árvore de diretórios | A árvore é necessária para preencher o campo "Arquivos" |
| Correção | `project.md` | Artefato atual (como está no disco), artefatos a montante, instruções, problema e decisão | Só se for design/tasks | — |

A saída de `openspec instructions ... --json` traz `context` (o resumo do `config.yaml`), `rules`, `template`, `instruction`, `resolvedOutputPath` e `dependencies`. Cole o JSON inteiro: `rules` e `template` são o que garante o formato.

---

## E0 – Preparação

1. Escolha a próxima change do plano em `docs/mapa-de-capacidades.md` e confirme que as changes de que ela depende já foram arquivadas (`openspec list`).
2. Crie a branch e a change:

```bash
git switch develop && git pull
git switch -c feature/[nome-da-change]
openspec new change [nome-da-change]
openspec status --change [nome-da-change]
```

3. Crie `docs/sdd/runs/[nome-da-change].md` com o cabeçalho: change, histórias, autor e data. Esse arquivo recebe o Resumo do Explore (E1), a tabela CA → Scenario (E3), o registro de execução (Passo 8) e a matriz de verificação (V3).
4. Avise no canal que a change é sua.

## Prompt base (colado no início de todo chat novo)

Arquivo: `docs/sdd/prompts/base.md`

```text
Você é o assistente de especificação do projeto PRO-ÁFRICA, que segue Spec-Driven
Development com OpenSpec (schema spec-driven).

Como esta conversa funciona (harness manual):
- Você só gera texto. Não executa comandos, não lê nem grava arquivos.
- Eu sou o harness: rodo a CLI `openspec`, leio arquivos e colo os resultados aqui.
- A skill OpenSpec anexada define a postura e o formato esperados. Onde ela mandar
  rodar comandos, usar AskUserQuestion, TodoWrite ou Task, mover arquivos ou gerar
  vários artefatos de uma vez, NÃO faça: trabalhe só na etapa pedida e me peça o
  que faltar.
- O `context` e as `rules` que vierem de `openspec instructions` são restrições
  para você. Siga-os, mas não os copie para o artefato.
- Se faltar arquivo ou informação, NÃO suponha. Responda apenas com uma linha por item:
  PRECISO DE: [arquivo ou informação] - motivo - basta a assinatura/trecho? sim/não
  e aguarde.
- Não invente requisitos. Decisões de negócio são da equipe: apresente opções e
  impactos, não escolha. Dúvidas vão para "Questões em aberto".
- Respeite as regras invioláveis do project.md.
- Escreva em PT-BR. Nomes de change e de capacidade exatamente como no mapa;
  identificadores de código em inglês.

[CONTEXTO GLOBAL: conteúdo de docs/project.md]
[SKILL DA ETAPA: conteúdo de SKILL-explore.md ou SKILL-propose.md]
```

---

## E1 – Explore

**Objetivo:** entender as histórias, levantar decisões de negócio e chegar a critérios de aceitação aprovados. Nada de arquitetura ou código.

**Entrada:** `base.md` (com `SKILL-explore.md`) + pacote E1.

### Prompt 1 – explorar

Arquivo: `docs/sdd/prompts/explore.md`

```text
ETAPA: explore · change [nome-da-change]

Histórias de usuário desta change:
[HISTÓRIAS: texto de cada US-xx copiado de docs/historias-de-usuario.md]

Mapa de capacidades e plano de changes:
[MAPA: conteúdo de docs/mapa-de-capacidades.md]

Specs atuais das capacidades envolvidas:
[SPECS PRINCIPAIS: conteúdo de openspec/specs/[capacidade]/spec.md, ou "nenhuma"]

Tarefa: explorar o problema antes de qualquer artefato. Não escreva proposal nem
spec, e não proponha arquitetura, classes, tabelas, telas detalhadas ou código.

Responda exatamente nesta estrutura:
1. Entendimento: em até 5 frases, o que as histórias pedem, para quem e por quê.
2. Fluxo do usuário: passos principais, incluindo erro, cancelamento e o que vem
   depois (diagrama ASCII se ajudar).
3. Decisões de negócio em aberto: numeradas D01, D02..., cada uma com 2 ou 3
   opções e o impacto de cada. Não escolha.
4. Ambiguidades: termos ou regras das histórias que admitem mais de uma leitura.
5. Critérios de aceitação candidatos: CA01, CA02..., cada um no formato
   "Dado / Quando / Então", com a US de origem. Inclua entrada inválida, item
   inexistente e cancelamento, não só o caminho feliz. Marque "(depende de Dxx)"
   quando o critério depender de uma decisão aberta.
6. Escopo: capacidade(s) do mapa afetada(s); o que fica fora desta change e em
   qual change do plano entra. Se a change parecer grande demais (mais de 4
   histórias, mais de uma capacidade ou critérios demais), diga e sugira a divisão.
7. Conflitos: algo que contradiz o project.md ou as specs atuais.
8. Questões em aberto: o que só a equipe/coordenação pode responder.
```

**Validação humana (antes do prompt 2):**

| O que conferir | Quem decide |
| --- | --- |
| Cada Dxx tem uma decisão e o motivo | Coordenação/equipe (negócio) |
| Cada CAxx foi aprovado, alterado ou removido; nenhum critério depende de decisão ainda aberta | Equipe |
| A capacidade bate com o mapa e a change cabe na regra de fatiamento | Dev + equipe |
| O LLM não inventou regra que não está nas histórias | Dev |

Se a resposta revelar que a change é grande demais, divida-a no mapa **antes** de seguir. Pode haver mais de uma rodada de perguntas e respostas no mesmo chat.

### Prompt 2 – fechar o Explore

Arquivo: `docs/sdd/prompts/explore-fechar.md`

```text
Respostas da equipe:
[RESPOSTAS: Dxx -> decisão e motivo; CAxx -> aprovado / alterado (novo texto) /
removido; respostas das questões em aberto]

Atualize a exploração com essas respostas e entregue SOMENTE o bloco abaixo, que
será salvo e usado nas próximas etapas. Não acrescente critério, decisão ou
capacidade que não esteja nas respostas ou na sua exploração já aprovada.

## Resumo do Explore - [nome-da-change]
### Histórias
### Entendimento
### Decisões tomadas
(Dxx: decisão - motivo)
### Critérios de aceitação aprovados
(CAxx - USxx - Dado / Quando / Então)
### Capacidades afetadas
(nome do mapa - nova ou existente)
### Fora de escopo
(item - change do plano onde entra)
### Questões em aberto
(as que continuam sem resposta e por que não bloqueiam esta change)
```

**Artefato:** cole o bloco na seção "Explore" de `docs/sdd/runs/[nome-da-change].md` e commite: `docs(sdd): explore [nome-da-change]`.

**Portão para o E2:** nenhuma questão em aberto que mude comportamento; todos os CA aprovados.

---

## E2 – Proposal

**Entrada:** `base.md` (com `SKILL-propose.md`) + pacote E2.

```bash
openspec instructions proposal --change [nome-da-change] --json
ls openspec/specs
```

Arquivo: `docs/sdd/prompts/proposal.md`

```text
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
```

**Validação humana:**

- [ ] Why cita todas as US da change; What Changes tem "Fora de escopo".
- [ ] Capabilities usa só nomes do mapa, com New/Modified correto.
- [ ] Nenhum comportamento que não esteja no Resumo do Explore.
- [ ] Impact sinaliza contrato de dados? Então o design vai exigir 2º dev.
- [ ] `openspec status --change [nome-da-change]` mostra proposal concluído.

Commit: `docs(openspec): proposal [nome-da-change]`.

---

## E3 – Specs

**Entrada:** `base.md` (com `SKILL-propose.md`) + pacote E3.

```bash
openspec instructions specs --change [nome-da-change] --json
```

Arquivo: `docs/sdd/prompts/specs.md`

```text
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
```

**Validação humana:**

- [ ] `openspec validate [nome-da-change] --strict` sem erros.
- [ ] Tabela "Cobertura CA": nenhum CA sem cenário e nenhum cenário sem CA ou decisão de origem.
- [ ] Todo requirement termina com `(USxx)` e tem cenário de erro quando a entrada pode ser inválida.
- [ ] Sem termos técnicos (`grep -niE "sqlite|qml|pyside|tabela|classe|json" openspec/changes/[nome-da-change]/specs` só deve achar nomes de contrato quando eles forem o próprio comportamento, como "exportar em JSON").
- [ ] MODIFIED preserva os cenários antigos que devem continuar valendo.
- [ ] A equipe leu os cenários: eles descrevem o que o usuário espera.

Salve a tabela "Cobertura CA" no `docs/sdd/runs/[nome-da-change].md`. Commit: `docs(openspec): specs [nome-da-change]`.

---

## E4 – Design

**Entrada:** `base.md` (com `SKILL-propose.md`) + pacote E4. Gere a árvore na hora (seção 5 do guia de contexto).

```bash
openspec instructions design --change [nome-da-change] --json
git ls-files --cached --others --exclude-standard -- . ':!openspec' ':!.gemini' | tree --fromfile --noreport
```

Arquivo: `docs/sdd/prompts/design.md`

```text
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
```

**Validação humana:**

- [ ] Nenhuma decisão viola as regras invioláveis (camadas, offline, formulário dinâmico).
- [ ] Formatos de arquivo registrados com alternativas.
- [ ] Toda decisão [2º DEV] foi aprovada por outra pessoa (registre o nome no `docs/sdd/runs/[nome-da-change].md`).
- [ ] Mudança em `config_schema.json` ou `annotation_output.json` está refletida na spec (seção 6 do `project.md`).
- [ ] Open Questions não escondem decisão de negócio; se esconderem, laço de correção para o E1/E3.

Commit: `docs(openspec): design [nome-da-change]`.

---

## E5 – Tasks

**Entrada:** `base.md` (com `SKILL-propose.md`) + pacote E5, com árvore nova.

```bash
openspec instructions tasks --change [nome-da-change] --json
```

Arquivo: `docs/sdd/prompts/tasks.md`

```text
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
```

**Validação humana:**

- [ ] Tabela "Cobertura" lista todos os cenários (compare com `grep -rn "#### Scenario" openspec/changes/[nome-da-change]/specs`).
- [ ] Cada task tem Cenários, Arquivos e Teste; nenhuma parece passar de ~1h ou tocar camadas demais.
- [ ] Testes ficam junto das tasks que implementam o comportamento.
- [ ] Ordem permite implementar e testar cada task sem depender de uma posterior.
- [ ] `openspec instructions apply --change [nome-da-change] --json` lista as tasks com a contagem esperada.

Commit: `docs(openspec): tasks [nome-da-change]`.

---

## E6 – Fechamento da especificação

```bash
openspec validate [nome-da-change] --strict
openspec status --change [nome-da-change]
git push -u origin feature/[nome-da-change]
```

- Os quatro artefatos aparecem como concluídos e a validação passa.
- Um 2º dev revisa os artefatos (pode ser um PR rascunho da `feature/[nome-da-change]`). Sem revisão, a Implementação não começa.
- Pronto: siga para o Passo 0 do Fluxo de Implementação.

---

## Laço de correção (usado por este e pelos outros fluxos)

Quando uma etapa posterior, a implementação ou um teste revela problema em um artefato:

1. **Identifique o artefato mais a montante afetado.** Ordem: Resumo do Explore → proposal → specs → design → tasks. Corrija ali, não a jusante.
2. **Decisão humana primeiro.** Se a correção muda comportamento, a equipe decide e a decisão entra no `docs/sdd/runs/[nome-da-change].md` (Dxx nova).
3. Rode `openspec instructions [artefato] --change [nome-da-change] --json` e use o prompt abaixo em chat novo.
4. Revise os artefatos a jusante que a correção afeta (ex.: spec mudou → conferir design e tasks).
5. `openspec validate [nome-da-change] --strict` e commit: `docs(openspec): corrige [artefato] de [nome-da-change]`.
6. Volte ao ponto em que estava (etapa de especificação, Passo 3 da Implementação ou V1 da Verificação).

Arquivo: `docs/sdd/prompts/correcao-artefato.md`

```text
ETAPA: correção de [artefato] · change [nome-da-change]

Problema encontrado: [o que está errado e onde apareceu: etapa, task N.M ou teste]
Decisão tomada pela equipe: [decisão, quem aprovou]

Artefato atual (exatamente como está no disco):
[CONTEÚDO DO ARTEFATO]

Artefatos a montante que ele deve respeitar:
[ex.: proposal.md e specs delta]

Instruções do OpenSpec (saída de `openspec instructions [artefato] --change [nome-da-change] --json`):
[INSTRUÇÕES]

Tarefa: altere somente o necessário para refletir a decisão, mantendo o formato
exigido pelas instruções. Não acrescente outras mudanças.
Entregue: o arquivo completo em um bloco markdown; a lista "Alterações" (o que
mudou e por quê); a lista dos artefatos a jusante que provavelmente precisam de
revisão, sem editá-los.
```

## Pedidos de contexto durante a especificação

- Formato único: `PRECISO DE: [arquivo ou informação] - motivo - basta a assinatura/trecho? sim/não`.
- Decida pela seção 7.2 do guia de contexto e responda com `RESULTADO DE leitura de [arquivo]:` ou com o modelo de negação da seção 7.3.
- Nas etapas E1 a E3, pedidos de código são negados: essas etapas não usam código.
- Limite de 2 rodadas por etapa. Passou disso, o pacote está incompleto ou a change está grande demais.

## Resumo das validações humanas

| Etapa | Validação obrigatória | Quem |
| --- | --- | --- |
| E1 | Decisões Dxx, critérios CAxx, tamanho da change | Coordenação/equipe |
| E2 | Escopo, fora de escopo, capacidades | Dev |
| E3 | Cobertura CA → Scenario, cenários de erro, `validate --strict` | Dev + equipe (leitura dos cenários) |
| E4 | Camadas, formatos de arquivo, decisões [2º DEV] | Dev + 2º dev |
| E5 | Cobertura de cenários, tamanho e ordem das tasks | Dev |
| E6 | Revisão dos quatro artefatos | 2º dev |
| Correção | Decisão que muda comportamento | Coordenação/equipe |

## Alterações de alinhamento (28/10/2026)

Pontos em que este fluxo complementa os demais documentos:

1. **Guia de contexto, seção 3:** a etapa `specs` passa a receber o Resumo do Explore (exigido pela regra "todo critério aprovado vira Scenario") e as specs principais das capacidades "Modified"; a etapa `tasks` passa a receber a árvore de diretórios (necessária para o campo "Arquivos"). Sugere-se atualizar a tabela do guia.
2. **Propose quebrado em etapas:** o `/opsx:propose` gera tudo de uma vez; aqui cada artefato tem seu chat e sua validação humana.
3. **Skills por etapa:** só um entre `SKILL-*` e `opsx-*` é enviado; `sync` e `archive` ficam com a CLI.
4. **Formato de tasks e tabela "Cobertura":** definidos no E5, consumidos pelo Passo 1 da Implementação e pelo V3 da Verificação.
5. **Marcador "(verificação manual)":** definido no E3, consumido pelo V3/V4 da Verificação.
6. **Branch:** criada no E0, como o Passo 0 da Implementação pressupõe.
