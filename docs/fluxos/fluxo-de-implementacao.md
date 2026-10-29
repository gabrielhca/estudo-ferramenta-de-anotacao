# Fluxo de Implementação

SDD com OpenSpec CLI · harness manual (agentless) · Tarefa 1 · Documento 1 de 2

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

0 Portão → 1 Escolher task → 2 Contexto → 3 Prompt → 4 Pedidos de contexto → 5 Revisão → 6 Aplicar → 7 Smoke run → 8 Registro.

Depois, segue o documento de Verificação e Finalização.

## Passo 0 – Portão de entrada

- Confirme que a change está pronta: `openspec validate [nome-da-change] --strict` sem erros e `openspec status --change [nome-da-change]` com proposal, specs, design e tasks concluídos.
- Os artefatos precisam estar revisados e commitados na branch `feature/[nome-da-change]`, criada a partir de `develop` no Fluxo de Especificação (workflow main/develop/feature).
- Faça `git switch feature/[nome-da-change]` e `git pull` antes de começar.

## Passo 1 – Escolher UMA task

- Rode `openspec instructions apply --change [nome-da-change] --json` e abra o `tasks.md`.
- Pegue a primeira task `- [ ]` e avise no canal que é sua, para duas pessoas não pegarem a mesma.
- A task deve caber em um PR pequeno. Se for grande, volte ao Fluxo de Especificação e quebre-a no `tasks.md`.
- Cada task já traz, entre parênteses, os **Cenários**, os **Arquivos** e o **Teste** que ela cobre. Use esses campos para montar o pacote do Passo 2.

## Passo 2 – Montar o pacote de contexto (nunca o projeto inteiro)

| Camada | O que enviar | Como obter |
| --- | --- | --- |
| 1. Global | `docs/project.md` (o `context` do `openspec/config.yaml` é um resumo dele) | Sempre, no início do chat |
| 2. Da mudança | `proposal.md`, spec delta da capability, `design.md`, texto da task atual e só os títulos das demais | Sempre |
| 3. De código | Árvore de diretórios; arquivos completos só dos que a task altera e suas dependências diretas; dos demais, só assinaturas (regras da seção 6 do guia de contexto) | Árvore: comandos da seção 5 do guia de contexto (abaixo). Assinaturas: `grep -nE "^(class\|def) " [arquivo]` |
| 4. De execução | `git log --oneline` das tasks concluídas, o registro de execução e, nas iterações, os erros de testes | Vazia na 1ª rodada |

Árvore de diretórios (gerar uma nova a cada task):

```bash
# Linux (recomendado)
git ls-files --cached --others --exclude-standard -- . ':!openspec' ':!.gemini' | tree --fromfile --noreport

# Windows (PowerShell)
git ls-files --cached --others --exclude-standard -- . ':!openspec' ':!.gemini'
```

Use um chat novo por task, colando o pacote, para não carregar contexto poluído. Nunca envie imagens, datasets, bancos `.db` ou pacotes `.zip`.

## Passo 3 – Enviar o prompt de implementação

```text
Você vai implementar UMA task de uma change OpenSpec.
[CONTEXTO GLOBAL: docs/project.md]
[CHANGE: proposal.md + spec delta + design.md]
[TASK ATUAL: texto da task [N.M], com Cenários, Arquivos e Teste]
[DEMAIS TASKS: só os títulos]
[CÓDIGO: árvore de diretórios + arquivos relevantes]
Regras:
- Implemente somente esta task. Não altere spec/design nem invente requisitos.
- Respeite as regras invioláveis do project.md.
- Entregue cada arquivo completo (ou diff unificado), com o caminho.
- Gere testes pytest para os cenários da spec cobertos por esta task, no arquivo indicado em "Teste".
- Liste suposições e ambiguidades em "Questões em aberto".
- Se faltar arquivo ou contexto, NÃO invente. Responda apenas com uma linha por item:
  "PRECISO DE: [arquivo ou símbolo] - motivo - basta a assinatura? sim/não"
  e aguarde.
```

## Passo 4 – Atender pedidos de contexto (o dev faz Read File / Search Code)

- Decida cada pedido pela ordem da seção 7.2 do guia de contexto: necessário para a task? viola regra inviolável? basta assinatura ou trecho? precisa do arquivo inteiro?
- Rode `grep -rn` ou `cat` e cole só o trecho necessário, precedido de `RESULTADO DE leitura de [arquivo]:`. Para negar, use o modelo da seção 7.3 do guia.
- Limite de 2 rodadas. Passou disso, o pacote de contexto está incompleto ou a task está grande demais: corrija e reinicie o chat.

## Passo 5 – Revisão humana antes de aplicar (permissões)

| Nível | Exemplos |
| --- | --- |
| Permitido | Criar ou editar arquivos dentro do escopo da task; criar testes |
| Exige confirmação de um 2º dev | Nova dependência; mudança de schema do banco; tocar arquivos fora do escopo; mudar assinatura pública usada por outras tasks; alterar `config_schema.json` ou `annotation_output.json` (contratos de dados) |
| Bloqueado | Editar proposal/spec/design/tasks fora do fluxo de especificação (exceção: marcar `- [x]` no V5 da Verificação); editar `openspec/specs/` à mão; push direto em main/develop; apagar arquivos que a task não criou |

As "Questões em aberto" do LLM são decisões de negócio. Se a resposta mudar o comportamento, volte à spec pelo laço de correção do Fluxo de Especificação, revalide com `openspec validate [nome-da-change] --strict` e só então continue. Não decida isso no código.

## Passo 6 – Aplicar (o dev faz Edit Code)

- Comece com `git status` limpo, para o `git diff` mostrar exatamente o que a task mudou.
- Crie ou edite os arquivos manualmente, a partir da resposta do LLM já revisada.

## Passo 7 – Smoke run

- Rode só os testes novos, por exemplo `pytest [arquivo-de-teste] -q`.
- Isso encerra a implementação e passa o bastão para a Verificação e Finalização. Se os testes nem rodarem, o erro entra no loop de correção (V2) daquele documento.

## Passo 8 – Registrar a execução

- Anote em `docs/sdd/runs/[nome-da-change].md`: task, autor, arquivos tocados, nº de iterações e decisões tomadas.
- Esse registro vira a camada "contexto de execução" da próxima task.

## Organização sugerida no repositório

```text
docs/
├── project.md                 # Contexto global (camada 1)
├── guia_de_contexto.md        # Fluxo de Gerenciamento de Contexto
└── sdd/
    ├── prompts/               # base.md, explore.md, proposal.md, specs.md, design.md, tasks.md,
    │                          # correcao-artefato.md, implement.md, fix-tests.md, verify.md
    ├── context/               # checklist do pacote de contexto
    └── runs/                  # [nome-da-change].md (registro de execução e matriz de verificação)
```

## Alterações de alinhamento (28/10/2026)

Ajustes feitos na conversão do PDF para que este fluxo converse com os demais:

1. **Validação estrita:** `openspec validate` passou a usar `--strict`, igual ao Fluxo de Especificação.
2. **Branch:** a `feature/[nome-da-change]` é criada no Fluxo de Especificação, porque os artefatos precisam ser commitados antes do Passo 0 e o push direto em `develop` é bloqueado.
3. **Contexto global:** a camada 1 é o `docs/project.md`, como define o guia de contexto; o `config.yaml` é só o resumo dele.
4. **Árvore de diretórios:** os comandos agora são os do guia de contexto, que excluem `openspec/` e `.gemini/` e respeitam o `.gitignore`.
5. **Pedido de contexto:** formato único `PRECISO DE: ... - motivo - basta a assinatura?`, usado também no Fluxo de Especificação; a decisão segue a seção 7.2 do guia.
6. **Permissões:** marcar `- [x]` no V5 é a única edição de `tasks.md` permitida fora da especificação; contratos de dados exigem 2º dev, conforme a seção 6 do `project.md`.
7. **Prompts:** a pasta `docs/sdd/prompts/` inclui os prompts do Fluxo de Especificação.
