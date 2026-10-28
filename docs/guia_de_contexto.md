# Guia de Gerenciamento de Contexto (LIPAI)
 
**Manual de Harness e Injeção de Contexto para LLMs**
 
## 1. Visão Geral
 
Este guia define como a equipe fornece informações do projeto PRO-ÁFRICA à IA. Como usamos o chat fora do editor, **o desenvolvedor atua como o Harness**: decide o que entra no contexto, executa os comandos e devolve os resultados.
 
**Princípio:** nunca enviar o projeto inteiro. Enviar o **mínimo necessário** para a task atual evita a degradação de contexto, pois a IA tende a esquecer regras ou inventar requisitos quando recebe informação demais.
 
## 2. As 4 Camadas de Contexto
 
| Camada | O que contém | Quando enviar |
| --- | --- | --- |
| **1. Global** | `docs/project.md` | Sempre, no início de cada conversa nova |
| **2. Da mudança** | `proposal.md`, `spec.md`, `design.md` e a **task atual** do `tasks.md` | Conforme a etapa (seção 3) |
| **3. De código** | Árvore de diretórios e arquivos relevantes para a task | Na etapa de implementação (seções 5 e 6) |
| **4. De execução** | Comando rodado, testes, erros, logs e alterações realizadas | Quando algo falhar (seção 8) |
 
## 3. Contexto por Etapa do Fluxo OpenSpec
 
A tabela abaixo indica **qual contexto** cada etapa do fluxo recebe.
 
| Etapa | Global | Da mudança | De código | De execução |
| --- | :---: | --- | :---: | :---: |
| `explore` | Sim | História de usuário | — | — |
| `proposal` | Sim | História + entendimento e decisões do *explore* + instruções do proposal | — | — |
| `specs` + `design` | Sim | `proposal.md` (e `spec.md`, no design) + instruções da etapa | Árvore no design | — |
| `tasks` | Sim | `proposal.md`, `spec.md`, `design.md` | — | — |
| `apply` | Sim | `spec.md`, `design.md` e **somente a task atual** | Árvore + arquivos da task | — |
| Correção de erro | — | Task atual | Arquivos alterados | Sim |
 
As instruções de cada artefato vêm do próprio OpenSpec:
 
```bash
openspec instructions proposal --change <nome-da-mudanca>
```
 
## 4. Contexto Global
 
- O contexto global é o arquivo `docs/project.md`: o que é o projeto, stack, camadas, contratos de dados e **regras invioláveis**.
- O campo `context:` do `openspec/config.yaml` deve conter um **resumo curto** do `project.md` (stack, camadas e regras principais).
- Sempre que o `project.md` mudar, atualize o `config.yaml` no mesmo commit, para que as duas fontes não divirjam.
- O `project.md` só precisa ser reenviado em **conversas novas**. Dentro da mesma conversa, não repita.
## 5. Geração da Árvore de Diretórios
 
A árvore permite que a IA entenda **onde cada coisa fica** sem ler o código de todos os arquivos. Rode os comandos na **raiz do projeto**.
 
### Linux
 
**Instalação:**
 
```bash
# Fedora
sudo dnf install tree
 
# Ubuntu/Debian
sudo apt install tree
```
 
**Opção recomendada** (respeita o `.gitignore`; só as pastas `openspec` e `.gemini` precisam ser excluídas manualmente):
 
```bash
git ls-files --cached --others --exclude-standard -- . ':!openspec' ':!.gemini' | tree --fromfile --noreport
```
 
**Opção manual** (filtrando pastas que não ajudam a IA):
 
```bash
tree -a --dirsfirst --noreport -I 'venv|.venv|__pycache__|.git|.gemini|openspec|.pytest_cache|node_modules'
```
 
### Windows
 
O `tree` do Windows é outro programa (não aceita `--fromfile` nem filtros), então os comandos acima não funcionam lá. No PowerShell, liste os arquivos com o Git:
 
```powershell
git ls-files --cached --others --exclude-standard -- . ':!openspec' ':!.gemini'
```
 
O resultado é uma lista de caminhos (ex.: `src/domain/project.py`), sem o desenho da árvore, o que já basta para a IA entender a estrutura. No `cmd`, troque as aspas simples por aspas duplas. Outra alternativa é usar o WSL e seguir os comandos da seção Linux.
 
### Regras
 
- Pastas de sistema, ambientes virtuais e o histórico do OpenSpec consomem contexto e não ajudam na implementação, portanto ficam de fora. Os artefatos OpenSpec chegam pela camada 2.
- É necessário **gerar uma árvore nova** a cada task, porque uma árvore antiga leva a IA a referenciar arquivos que não existem mais.
- Quando o projeto ficar grande, limite a profundidade (`-L 3`, no Linux) ou gere só a parte relevante (ex.: `tree src/services` no Linux, ou `git ls-files --cached --others --exclude-standard src/services` no Windows).
- Cole o resultado no chat ao iniciar a implementação de uma história de usuário (ex.: *US01 — Criar projeto*).
## 6. Quando Enviar Arquivos Completos
 
**Regra:** envie o código-fonte **integral** de um arquivo apenas se ele for **modificado** ou **consumido diretamente** pela task atual. Todo o resto fica de fora.
 
| Situação do arquivo | O que enviar |
| --- | --- |
| Será criado ou alterado pela task | Arquivo **inteiro** se já existir |
| Define algo que a task usa | Arquivo inteiro, se pequeno; senão, só a classe ou estrutura |
| Apenas é chamado pela task | **Só a assinatura** da função/classe |
| Teste que cobre o que será alterado | Arquivo de teste correspondente |
| Não tem relação com a task | **Não enviar** |
 
Também **nunca envie** imagens, datasets, bancos `.db` ou pacotes `.zip`. Se a IA precisar saber o formato, descreva a estrutura.
 
## 7. Tratamento de Pedidos por Contexto Adicional
 
Durante a implementação, a IA pode pedir mais arquivos do que o necessário. O desenvolvedor controla o *harness* e define os limites.
 
### 7.1 Instrução fixa nos prompts de implementação
 
Inclua em todo prompt de `apply`, para que a IA peça contexto de forma controlada em vez de supor:
 
```text
Use apenas o contexto fornecido. Se precisar de mais informação, NÃO suponha
o conteúdo de arquivos que não recebeu. Liste o que precisa: caminho do arquivo,
o motivo e se basta a assinatura. Aguarde minha resposta antes de continuar.
```
 
### 7.2 Como decidir cada pedido
 
Para cada arquivo que a IA pedir, siga esta ordem:
 
1. **É estritamente necessário para a task atual?** Se não, negue e lembre a task.
2. **Viola uma regra inviolável do `project.md`?** Se sim, negue citando a regra.
3. **Basta a assinatura ou um trecho?** Envie só isso, nunca as 500 linhas do arquivo.
4. **Precisa mesmo do arquivo inteiro e ele é consumido ou modificado pela task?** Envie.
5. **Os pedidos estão se repetindo?** É sinal de que a task está grande demais ou de que a spec/design está incompleta. Pare, divida a task ou corrija o artefato OpenSpec, em vez de enviar cada vez mais arquivos.
### 7.3 Modelos de resposta
 
**Negar:**
 
```text
Não enviarei `<arquivo>`: não é necessário para a task atual (<descrição da task>).
Prossiga com o contexto já fornecido. Se ainda faltar algo, descreva exatamente
a informação de que precisa, não o arquivo.
```
 
**Envio parcial:**
 
```text
Segue apenas a assinatura que você precisa chamar:
<assinatura da função ou estrutura da classe>
```
 
## 8. Contexto de Execução
 
A IA não enxerga o terminal. Quando algo falhar, o desenvolvedor deve devolver o contexto exato. Não diga apenas "quebrou": informe o comando, o erro completo e o que foi alterado.
 
**Modelo de prompt de correção:**
 
```text
A execução dos testes falhou.
Comando rodado: pytest
Alterações realizadas: <arquivos alterados nesta task>
Log do terminal:
<cole aqui o traceback e o teste que falhou>
Analise o erro com base nas regras do project.md e corrija somente a task atual.
```
 
Em logs muito longos, envie o teste que falhou e o *traceback*, não a saída inteira.
 
## 9. Organização no Repositório
 
```text
docs/
├── project.md              # Contexto global (camada 1)
└── guia_de_contexto.md     # Este guia
openspec/
└── config.yaml             # Resumo do project.md no campo "context"
```
 
## 10. Checklist Antes de Enviar um Prompt
 
- [ ] Conversa nova? Enviei o `project.md`.
- [ ] Enviei apenas o contexto da **etapa atual** (seção 3)?
- [ ] A árvore de diretórios é recente?
- [ ] Os arquivos enviados são os que a task modifica ou consome (seção 6)?
- [ ] O prompt de implementação traz a instrução de pedir contexto em vez de supor (seção 7.1)?
 