---
name: rina-traduzioni-le-costano-20s
description: "La catena traduzioni del ramo 'da richiedere' era in serie e riapri' il timeout il 21/10; risolta lo stesso giorno agganciandola a Componi clausole"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T11:16:32.536Z
---

## Il problema (mattina del 21/10)

La catena aggiunta per tradurre il ramo **"da richiedere"** (difetto 2419) —
`Clausole traduzioni LE` → `Lista traduzioni LE` → `Coppie traduzioni LE` → `Mappa traduzioni LE` —
partiva dall'**output** di `Lista altre LE`, quindi si sommava ai suoi 36 secondi.
Risultato: `FlowActionTimedOut` sul caso di regressione standard.

**La lezione che resta valida.** Il 2419 era stato chiuso verificando **solo** che la traduzione
uscisse giusta. **Dopo ogni azione aggiunta al flusso va rimisurata la durata totale della run**,
non solo l'output.

## La soluzione (pomeriggio del 21/10) — applicata e pubblicata

**L'intuizione sta in una riga di codice**: `Lista altre LE` filtra su
`body('Componi_clausole')`, cioe' cerca cross-company **tutti** i codici candidati, non solo
quelli mancanti. Quindi i codici servono alla catena traduzioni sono gia' noti subito dopo
`Componi clausole`: la dipendenza da `Filtra_non_disponibili` era di comodo, non necessaria.

| azione | modifica |
|---|---|
| `Clausole traduzioni LE` From | `@body('Componi_clausole')` |
| `Clausole traduzioni LE` Map | `@replace(item(), 'ItemNumber eq ', 'ProductNumber eq ')` |
| `Clausole traduzioni LE` Run after | `Componi clausole` (era `Filtra non disponibili`) |
| `Lista traduzioni LE` Top Count | 500 (era 200) |
| `Seleziona non disponibili` Run after | `Mappa traduzioni LE` **+** `Filtra non disponibili` |

L'ultima riga tiene il grafo connesso: staccata la catena, `Filtra non disponibili` sarebbe
rimasto senza successori.

**Il `Run after` si modifica dalla scheda Settings dell'azione.** Non serve il drag and drop, che
in questo designer non funziona: vedi [[power-automate-espressioni-non-si-salvano]].

**Guadagno:** 1m16s → **1m01s**. Sommato a [[rina-express-mode-attivo]]: da 1m42s fallita a
1m01s riuscita.

**Verificato funzionalmente**, non solo sui tempi: su RSPA, domanda in francese, l'output di
`Seleziona non disponibili` passa da `Fornitura ricambi per attrezzature` a
`Fourniture de pièces de rechange pour équipements`.

**Perche' 15s e non 20:** `Lista traduzioni LE` ora dura 34s (riceve tutti i codici), ma gira in
parallelo a `Lista categorie` che dura anch'essa 34s ed e' sul percorso critico comunque. La
catena e' diventata gratuita.

## Quello che resta (piu' grosso di quello risolto)

Su **RSPA** il flusso impiega ancora 1m41s e il client Copilot molla, anche se **il flusso riesce**.
La causa e' la catena sequenziale del ramo destro: `Lista categorie` → `Seleziona nomi categoria`
→ `Componi clausole assegn` → `Lista assegnazioni` → `Componi clausole prodotti` →
`Lista articoli` → `Lista nature`. Sei query in fila. E' preesistente.
