# Projeto PRO-ÁFRICA (LIPAI)
 
**Ferramenta de Curadoria e Anotação de Imagens**
 
> Contexto global do projeto. Este arquivo é enviado no início de **toda** sessão de trabalho (explore, proposal, specs, design, tasks e apply). Você deve tratá-lo como fonte de verdade sobre o que é o projeto e quais regras não podem ser quebradas.
 
## 1. Visão Geral
 
Aplicação desktop **offline-first** para curadoria de imagens e anotação (classificação e *bounding boxes*) com suporte a múltiplos especialistas. Perguntas e classes de objetos são definidas pelo coordenador, nunca fixadas no código. Os dados coletados alimentarão *datasets* auditáveis para modelos de *Deep Learning*.
 
## 2. Papéis
 
- **Coordenador:** cria o projeto de anotação, adiciona imagens, configura perguntas e classes, gera o pacote (.zip) para distribuição, importa as anotações dos especialistas, resolve divergências e exporta o dataset final.
- **Especialista:** abre o pacote recebido, anota as imagens (respostas, observações e *bounding boxes*) sem depender de internet e exporta suas anotações (.json).

## 3. Fluxo de Dados
 
1. **Configuração e empacotamento (Coordenador):** seleciona as imagens, define perguntas e classes e gera o **Pacote de Projeto (.zip)**.
2. **Distribuição e execução offline (Especialistas):** cada especialista abre sua cópia do pacote e realiza as marcações.
3. **Exportação individual:** a ferramenta gera um **Arquivo de Anotações (.json)** por especialista.
4. **Agregação e consenso (Coordenador):** importa os .json, aponta divergências entre especialistas e registra a decisão final.

## 4. Stack Tecnológica
 
- **Linguagem:** Python
- **Interface gráfica:** PySide6 / QML
- **Banco de dados:** SQLite (local)
- **Testes:** pytest

## 5. Arquitetura em Camadas
 
- **UI:** componentes visuais em QML e controladores PySide6. Apenas apresenta dados e captura ações do usuário.
- **Services:** orquestram os casos de uso e fazem a ponte entre a UI e o Domain.
- **Domain:** modelos de negócio, entidades e validações puras, sem dependência de PySide6, SQLite ou sistema de arquivos.
- **Repositories:** interfaces de acesso a dados.
- **Storage:** implementação concreta dos Repositories em SQLite e a manipulação de arquivos, incluindo a leitura e escrita de pacotes ZIP e arquivos JSON.
**Direção das dependências:** UI → Services → Domain e Repositories. O Storage implementa os Repositories. O Domain não depende de nenhuma outra camada.
**Entidades principais:** Projeto, Imagem, Pergunta (com opções de resposta, obrigatoriedade e seção), Classe de objeto, Especialista, Anotação, BoundingBox e Pacote.
 
## 6. Contratos de Dados
 
Os arquivos abaixo são a ponte entre o módulo do Coordenador e o módulo do Especialista. Nenhuma mudança neles deve ser feita sem atualizar a spec correspondente:
 
- `config_schema.json`: configuração do projeto com perguntas, respostas, regras de obrigatoriedade, classes e instruções, gerada pelo coordenador e usada para renderizar o formulário dinâmico.
- `annotation_output.json`: respostas e coordenadas de cada especialista.
- **Coordenadas de *bounding box*:** X, Y, largura e altura **normalizados no intervalo de 0 a 1**.

## 7. Regras Invioláveis
 
1. **Offline-first:** o sistema não pode depender de internet, APIs em nuvem, CDNs ou telemetria em nenhuma funcionalidade.
2. **Isolamento de banco de dados:** somente a camada Storage acessa o SQLite. A UI e os Services NUNCA o acessam diretamente; toda persistência passa obrigatoriamente pelos Repositories.
3. **Isolamento de regras:** regras de negócio não ficam na UI. Ficam no Domain e são orquestradas pelos Services.
4. **Formulário dinâmico:** perguntas, opções de resposta e classes vêm da configuração do coordenador. Nunca escreva no código perguntas ou campos específicos de um domínio.
5. **Idioma:** todos os artefatos (documentação OpenSpec, comentários, docstrings, mensagens de erro e textos da interface) em Português (PT-BR). Identificadores de código (nomes de classes, funções e arquivos) seguem a convenção Python (PEP 8) em inglês.
6. **Contexto limitado:** se faltar informação, liste os arquivos de que precisa e por quê, em vez de supor o conteúdo de arquivos que não foram enviados. Dúvidas de negócio devem ser registradas como ‘Questões em aberto’, sem inventar requisitos.
