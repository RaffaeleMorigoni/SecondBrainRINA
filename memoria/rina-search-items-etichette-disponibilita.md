---
name: rina-search-items-etichette-disponibilita
description: Search Items etichetta i codici come disponibili o da richiedere; il connettore F&O non accetta crossCompany via JSON
metadata:
  type: project
---

Dal 2026-09-29 il flusso `RINA - Search Items` (`f9875f51-7db3-f111-aaac-70a8a537aa21`) restituisce
due liste distinte: `Items` (rilasciati nella LE dell'utente) e `OtherLegalEntities` (pertinenti ma
non rilasciati, con l'elenco delle LE dove esistono). Azioni aggiunte: `Codici_disponibili`,
`Lista_altre_LE`, `Filtra_non_disponibili`, `Seleziona_non_disponibili`. Il `$top` per keyword e
passato da 10 a 40 e `Lista_articoli` da 100 a 200.

**Why:** con `$top 10` il taglio era **alfabetico per codice**, non per pertinenza: per
"manutenzione" si fermava a SE105002 e scartava SE152002.0001, il codice corretto. Era questa, non
il filtro per legal entity, la causa del caso segnalato dai tester.

**How to apply:** il connettore Fin & Ops **non accetta piu il parametro `crossCompany` inserito via
JSON** (il flow checker risponde "no longer present in the operation schema"). Va impostato dalla UI
dell'azione, parametro avanzato **"Cross Company" = Yes**; un'azione incollata via JSON va poi anche
ricollegata con "Change connection reference". Vedi [[rina-rspa-anagrafica-mmd-vuota]] e
[[copilot-studio-browser-quirks]].

**Nota sulle istruzioni del child agent:** una regola aggiunta in coda alle istruzioni viene
ignorata se una riga precedente la contraddice. La riga del blocco ITEM SEARCH diceva ancora
"the tool returns only items released in the user's company" e il modello seguiva quella,
scartando OtherLegalEntities pur ricevendolo. Serviti tre interventi: correggere la riga
contraddittoria, e mettere la regola come **prima riga del blocco ITEM SEARCH**
("MANDATORY TWO GROUP ANSWER"), non in fondo al testo. Solo allora la prima risposta e uscita
divisa nei due gruppi.

Per modificare una riga in mezzo alle istruzioni: `execCommand('insertText')` sull'intero testo
non funziona (React lo scarta). Funziona selezionare via Range il testo esatto da sostituire e
poi digitare con la tastiera reale.
