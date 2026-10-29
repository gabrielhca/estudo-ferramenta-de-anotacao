# Histórias de Usuário

> Fonte: Histórias de usuários.docx. Este arquivo é a versão versionada no repositório;
> as histórias são copiadas daqui pelo ID para o prompt da LLM (ver Fluxo de Especificação).

## Criação e gerenciamento do projeto de anotação

### US01 — Criar projeto de anotação
Como coordenador, quero criar um projeto de anotação para organizar um novo dataset.

### US02 — Informar dados do projeto
Como coordenador, quero definir nome, descrição e instruções do projeto para identificar e orientar a anotação.

### US03 — Adicionar imagens
Como coordenador, quero adicionar imagens ao projeto para que elas possam ser anotadas.

### US04 — Remover imagens
Como coordenador, quero remover imagens do projeto para corrigir o conjunto de dados antes da distribuição.

## Configuração da anotação

### US05 — Criar perguntas
Como coordenador, quero criar perguntas de anotação para coletar informações sobre cada imagem.

### US06 — Configurar respostas
Como coordenador, quero definir as opções de resposta de uma pergunta para padronizar as avaliações.

### US07 — Organizar perguntas
Como coordenador, quero organizar as perguntas em seções para facilitar o preenchimento pelo especialista.

### US08 — Configurar classes de objetos
Como coordenador, quero definir classes de objetos para que possam ser utilizadas nas marcações das imagens.

## Pacote do dataset

### US09 — Gerar pacote
Como coordenador, quero gerar um pacote do projeto para distribuí-lo aos especialistas.

### US10 — Abrir pacote
Como especialista, quero abrir um pacote recebido para iniciar a anotação do dataset.

## Anotação das imagens

### US11 — Identificar usuário
Como usuário (coordenador ou especialista), quero acessar a aplicação informando meu nome e meu papel para que minhas ações e anotações sejam associadas a mim.

### US12 — Visualizar imagens
Como especialista, quero navegar pelas imagens do dataset para realizar as anotações.

### US13 — Responder perguntas
Como especialista, quero responder às perguntas configuradas para registrar minha avaliação de cada imagem.

### US14 — Registrar observações
Como especialista, quero adicionar observações a uma imagem para complementar minha avaliação.

### US15 — Salvar progresso
Como especialista, quero que meu progresso seja salvo para continuar a anotação posteriormente.

### US16 — Acompanhar progresso
Como especialista, quero visualizar quais imagens já foram anotadas e quais ainda estão pendentes.

## Bounding boxes

### US17 — Criar bounding box
Como especialista, quero desenhar uma bounding box sobre uma imagem para identificar um objeto.

### US18 — Classificar bounding box
Como especialista, quero associar uma classe a uma bounding box para identificar o objeto marcado.

### US19 — Editar bounding box
Como especialista, quero alterar uma bounding box existente para corrigir uma marcação.

### US20 — Remover bounding box
Como especialista, quero remover uma bounding box para excluir uma marcação incorreta.

## Exportação das anotações

### US21 — Exportar anotações
Como especialista, quero exportar minhas anotações para enviá-las ao coordenador.

### US22 — Exportar detecções
Como especialista, quero exportar as marcações em formato adequado para utilização em modelos de detecção.

## Agregação e consenso

### US23 — Importar avaliações
Como coordenador, quero importar os arquivos de anotação dos especialistas para reunir suas avaliações.

### US24 — Agregar avaliações
Como coordenador, quero combinar as avaliações dos especialistas para analisar o dataset.

### US25 — Identificar divergências
Como coordenador, quero visualizar os casos em que os especialistas discordaram para direcionar a revisão.

### US26 — Revisar divergências
Como coordenador, quero revisar casos divergentes para definir uma decisão final.

### US27 — Registrar consenso
Como coordenador, quero registrar a decisão final de um caso para construir a anotação consolidada.

### US28 — Exportar dataset final
Como coordenador, quero exportar as anotações consolidadas para obter o dataset final.

### US29 — Gerar relatório
Como coordenador, quero gerar um relatório da anotação para acompanhar participação, concordância e divergências.