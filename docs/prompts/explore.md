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
