---
name: rina-doppio-messaggio-architettura
description: Il doppio messaggio e la label MMD mancante si risolvono nel Response formatting dell'orchestratore, non nelle istruzioni del figlio
metadata:
  type: project
---

Doppio messaggio ed etichetta `MMD |` mancante avevano un'unica causa: a comporre quelle
risposte non era l'agente figlio ma **l'orchestratore di progetto** `Copilot for finance
and operations apps` (`679d8694-1250-4780-946e-6db1c2109ae1`).

**La soluzione, trovata il 09/10/2026:** scrivere le regole di forma nel campo
*Response formatting* **dell'orchestratore** (Settings → Generative AI → Responses,
500 caratteri). Al testo originale di Chiara e' stato appeso
` Inizia ogni risposta con "MMD | ".`. Risultato in F&O: 7 turni consecutivi, una sola
risposta ciascuno, label sempre presente. Task DevOps 2313 e 2333 chiusi.

**Le vie che NON funzionano**, gia' provate e da non ritentare:
- rincorrere il difetto nelle istruzioni del figlio (4 tentativi, torna in forma nuova);
- scheda Outputs → *Completion* → *After running* = `Send specific response` con
  `Output.resolvedResult`: la risposta arriva corretta e singola, ma l'orchestratore
  continua a voler parlare e produce `Empty response from AI model -
  ContentValidationError` dopo ogni risposta (provato il 02/10, ripristinato).

**Come applicarlo:** per qualunque difetto di forma che sopravvive alle istruzioni del
figlio, guardare prima l'orchestratore. Vedi [[rina-formato-fo-non-governabile]] e
[[rina-suite-regressione]].

**RIAPERTO il 15/10.** Dopo lo spostamento della ricerca sull'orchestratore il doppio messaggio
e' tornato: "MMD per manutenzione degli edifici..." in F&O produce **due blocchi `MMD |`
separati**, uno per articolo, piu' un terzo paragrafo di commento. Il task DevOps 2333 era stato
messo a Done il 09/10 e va riaperto.

