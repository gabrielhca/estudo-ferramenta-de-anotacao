# Mapa de Capacidades e Plano de Changes

> **Status: PROPOSTA, a validar na reunião.**
> Como cada feature é especificada em uma conversa isolada do Claude, este mapa é o que
> mantém o vocabulário consistente entre conversas. Ele vai para o conhecimento do
> projeto no Claude e é citado no `openspec/config.yaml`.

## Capacidades (specs permanentes em `openspec/specs/<capacidade>/spec.md`)

Uma **capacidade** é um comportamento durável do sistema. Ela não é uma tarefa. Ela acumula
requisitos ao longo de várias changes.

| Capacidade | O que cobre | Histórias |
|---|---|---|
| `projeto-anotacao` | Criar o projeto e manter nome, descrição e instruções | US01, US02 |
| `imagens-projeto` | Adicionar e remover imagens do projeto | US03, US04 |
| `perguntas-anotacao` | Perguntas, opções de resposta e seções | US05, US06, US07 |
| `classes-objeto` | Classes usadas nas bounding boxes | US08 |
| `pacote-dataset` | Gerar e abrir o pacote distribuído aos especialistas | US09, US10 |
| `sessao-especialista` | Identificar o especialista, salvar e retomar o progresso | US11, US15, US16 |
| `anotacao-imagens` | Navegar, responder perguntas e registrar observações | US12, US13, US14 |
| `bounding-boxes` | Criar, classificar, editar e remover caixas | US17–US20 |
| `exportacao-anotacoes` | Exportar anotações do especialista e detecções | US21, US22 |
| `consenso-avaliacoes` | Importar, agregar, apontar divergências, desempatar e consolidar | US23–US27 |
| `dataset-final` | Exportar o dataset consolidado | US28 |
| `relatorio-anotacao` | Relatório de participação, concordância e divergência | US29 |

## Plano de changes (ordem sugerida)

Regra de fatiamento: **uma change entrega de 1 a 4 histórias da mesma capacidade e
termina em um comportamento testável.** Se o Explore revelar que a change ficou
grande, divida-a antes da proposta.

| # | Change (nome kebab-case) | Histórias | Depende de |
|---|---|---|---|
| 1 | `criar-projeto-anotacao` | US01, US02 | - |
| 2 | `gerenciar-imagens-projeto` | US03, US04 | 1 |
| 3 | `configurar-perguntas` | US05, US06, US07 | 1 |
| 4 | `configurar-classes-objeto` | US08 | 1 |
| 5 | `gerar-abrir-pacote` | US09, US10 | 2, 3, 4 |
| 6 | `sessao-especialista` | US11, US15, US16 | 5 |
| 7 | `anotar-imagens` | US12, US13, US14 | 6 |
| 8 | `bounding-boxes` | US17, US18, US19, US20 | 7 |
| 9 | `exportar-anotacoes` | US21, US22 | 7, 8 |
| 10 | `importar-agregar-avaliacoes` | US23, US24 | 9 |
| 11 | `resolver-divergencias` | US25, US26, US27 | 10 |
| 12 | `exportar-dataset-final` | US28 | 11 |
| 13 | `relatorio-anotacao` | US29 | 12 |
