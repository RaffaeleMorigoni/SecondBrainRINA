---
name: rina-stile-commenti-devops
description: "I commenti su DevOps vanno brevi e in prima persona, come li scriverebbe Raffaele: niente tabelle, niente relazioni"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T13:41:15.818Z
---

Sui work item DevOps (e in generale) i commenti devono essere **corti e in linguaggio umano**,
scritti come se li scrivesse Raffaele di suo pugno.

**Si':** prima persona ("ho corretto", "ho riprovato oggi"), due o tre capoversi, la conclusione
per prima, il dettaglio tecnico solo se serve a chi legge.

**No:** tabelle, titoli in maiuscolo tipo "ESITO VERIFICATO", elenchi di sostituzioni riga per
riga, ricostruzioni lunghe del ragionamento, formule da relazione.

**Why:** i commenti li leggono Chiara, Giorgio, Andrea e i tester durante il SAL. Un muro di testo
non viene letto, e la parte che conta si perde. Una cosa scritta in tre righe viene letta.

**How to apply:** scrivere il commento, poi togliere tutto quello che non cambia cosa fa chi
legge. Il dettaglio completo sta nei file di `plans/`, non sul task.

Vale anche per i messaggi a colleghi e per le risposte in chat. Collegato a
[[rina-devops-significato-stati]] e [[rina-task-descrizione-non-registro]].
