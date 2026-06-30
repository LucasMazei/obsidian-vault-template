---
tags:
  - goal
type: Espiritual
current: 0
target: 1
year: 2026
direction: Maximize
---
# O Alvo
_Escrever um livro de 200 páginas até Dez/2026._

# A Identidade
_Eu sou um escritor que prioriza a arte sobre o entretenimento._

# O Sentimento
_Quero sentir a satisfação de ter minhas ideias organizadas e perpetuadas._

# O Preço a Pagar
_Vou acordar 1h mais cedo todos os dias e recusar convites nas manhãs de sábado._

# O mínimo Viável
_Nos dias ruins, escreverei ao menos 1 parágrafo (regra do não-zero)._

# Ações (Sistemas)

```dataview
TABLE without id
link(file.name) as "Ação",
status
FROM #action 
WHERE contains(file.outlinks, this.file.link)
```
