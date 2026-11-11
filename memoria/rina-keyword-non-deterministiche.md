---
name: rina-keyword-non-deterministiche
description: "Le parole chiave estratte cambiano a ogni esecuzione sulla stessa domanda: e' la causa a monte dell'intermittenza inseguita tutto il 30/10"
metadata:
  type: project
---

**30/10/2026.** Stessa domanda portoghese, stesso agente, stesso modello, venti minuti di
distanza:

| run | parole passate a `Filtra parole` |
|---|---|
| 17:09 | `certificazione`, **`ISO`**, `servizio` |
| 17:30 | `certificazione`, `sanita`, `sociale` |

**L'estrazione delle keyword non e' deterministica.** La ricerca trova o non trova a seconda di
quali parole il modello ha sorteggiato quel giro.

**Questo spiega l'intermittenza di tutta la giornata** e rende inutili tre diagnosi date prima:
il caso francese che alle 16:36 trovava sei codici e alle 16:45 nessuno
([[rina-ricerca-francese-non-trova]]) non dipendeva dalla query, dal carico o dalla lingua.

**Conseguenza operativa**: prima di attribuire un esito alla query, al modello o alla lingua,
**leggere `Filtra parole` di quella run**. Due run della stessa domanda non sono confrontabili
se le parole differiscono - ed e' il caso normale, non l'eccezione.

E' un difetto **piu' a monte** del 2494 ([[rina-search-items-query-sequenziali]]) e di
[[rina-codici-fuori-dalla-ricerca]]. Finche' resta, ogni misura su questa catena e' un sorteggio.

Dettaglio in `plans/20261030_caso_PS101002_0060_portoghese.md`.
