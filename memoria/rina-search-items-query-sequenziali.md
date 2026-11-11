---
name: rina-search-items-query-sequenziali
description: "Il collo di bottiglia di Search Items e' Lista nature (50s su 96), non le quattro ricerche per parola, che sono gia' parallele"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-08T14:28:49.594Z
---

Sulla run dell'08/10/2026 alle 11:47 — quella del `FlowActionInternalServerError` segnalato da
Emanuela su "sviluppo di soluzioni informatiche... capex" — il flusso `RINA - Search Items`
(`f9875f51-7db3-f111-aaac-70a8a537aa21`) ha impiegato **96 secondi** e il grosso sta in una sola
azione:

| Azione | Durata |
|---|---|
| Lista utente | 18s |
| Lista prodotti 1-4 (in parallelo) | 22-23s |
| Lista articoli | 4s |
| **Lista nature** | **50s** |
| tutto il resto | ~0s |

**Le quattro ricerche per parola sono gia' parallele** — gli edge vanno tutti da `Filtra_parole`
alle quattro `Lista_prodotti` e poi a `Componi_clausole`, e nel grafico della run appaiono
affiancate. Avevo dedotto il contrario sommando le durate: **sbagliato**, i tempi vanno letti sul
grafico o sui timestamp, non sommati.

**Perche' `Lista nature` costa 50 secondi.** Confronto con `Lista articoli`, che fa un lavoro
analogo in **4 secondi**:

- `Lista articoli`: entita' `ReleasedProductsV2`, filtro `dataAreaId eq 'rsse' and (17 OR su
  ItemNumber)`, `$select` con sei campi → 4s.
- `Lista nature`: entita' `ProductsV2`, filtro
  `AVY0587_PurchaseNature eq ...ItemPurchaseNature'Hybrid' and (16 OR su ProductNumber)`,
  `$select` solo `ProductNumber`, `$top` 100 → 50s.

Stesso numero di clausole, `$select` piu' leggero, dodici volte piu' lento: il costo e' la
condizione sul **campo custom** `AVY0587_PurchaseNature` su `ProductsV2`, che evidentemente non e'
indicizzato e costringe a una scansione prima di applicare il filtro per chiave.

**Fix proposto (non ancora applicato):** togliere la condizione sulla nature dal `$filter`,
lasciando i soli OR per `ProductNumber`, e aggiungere `AVY0587_PurchaseNature` al `$select`,
spostando il filtro Hybrid in memoria nelle azioni che seguono (`Seleziona nature`, `Filtra
hybrid`). La query torna una lookup per chiave come quella da 4 secondi. Tocca tre azioni, quindi
va provato prima su bozza e misurato sulla stessa domanda.

Vedi [[rina-search-items-etichette-disponibilita]].

**08/10/2026 - la scorciatoia e' esclusa: il campo non sta su ReleasedProductsV2.** Provato ad
aggiungere `AVY0587_PurchaseNature` al `$select` di `Lista articoli` per eliminare del tutto
`Lista nature`: la run fallisce in 2 secondi con `Action 'Lista_articoli' failed`. Il campo
esiste solo su `ProductsV2`. Il `$select` e' stato ripristinato e il flusso ripubblicato (run di
verifica Succeeded in 5s). **Non riprovare questa strada.**

Resta quindi il piano sul filtro in memoria: togliere dal `$filter` di `Lista nature` la sola
condizione sulla nature (la parte fissa `AVY0587_PurchaseNature eq ...'Hybrid' and `), aggiungere
il campo al suo `$select`, e applicare il filtro Hybrid dopo, dove oggi c'e' il `Select`
`Seleziona nature`. Il punto di ripristino con le definizioni originali e' in
`scratchpad/ROLLBACK_search_items_lista_nature.md`.

**08/10/2026 - FIX APPLICATO E PUBBLICATO: da 96 a 23 secondi.** Tre modifiche al flusso:

1. `Lista nature` — `$filter` ridotto ai soli OR per ProductNumber:
   `@concat('(', if(empty(body('Componi_clausole_nature')), 'ProductNumber eq ''ZZNONEZZ''', join(body('Componi_clausole_nature'), ' or ')), ')')`
   e `$select` esteso a `ProductNumber,AVY0587_PurchaseNature`.
2. Nuova azione **`Filter array`** fra `Lista nature` e `Seleziona nature`:
   From `@outputs('Lista_nature')?['body/value']`, condizione
   `@contains(string(item()?['AVY0587_PurchaseNature']), 'Hybrid')` — scritta con `contains` su
   `string()` per non dipendere dal formato con cui l'enum torna nel JSON.
3. `Seleziona nature` — `from` ripuntato a `@body('Filter_array')`.

**Esito verificato** sulla domanda di Emanuela ("sviluppo di soluzioni informatiche... capex"):
run **Succeeded in 23 secondi** (prima 96, con errore), risposta corretta con `SE302501.0001` e
`SE707005.0001`, le due sezioni e la richiesta di scegliere. La segnalazione Alta del registro
UAT dell'8 ottobre e' chiusa.

**La lezione sul metodo:** una condizione fissa su un campo custom dentro il `$filter` costa
dieci volte il filtro per chiave. Quando serve filtrare su un campo custom di F&O, portare la
chiave nel `$filter` e il campo custom in un `Filter array` a valle.

## 15/10: Cross Company raddoppia i tempi e fa sforare Express mode

Dopo aver messo `Cross Company = Yes` su `Lista_articoli` (necessario, vedi
[[rina-company-sessione-dataareaid]]) i tempi sono passati da ~31s a **69s** sul caso di
regressione, e una ricerca ampia ("articoli per il laboratorio") e' finita in **TimedOut a
2 minuti esatti**.

Il messaggio del portale e' esplicito: *"Your flow timed out. Flows running in Express mode
must complete within 2 minutes."* Il flusso ha **Express mode** attivo nel trigger, quindi il
tetto e' 120 secondi netti, non negoziabile dall'interno del flusso.

In F&O l'utente vede `FlowActionTimedOut` e il messaggio "Il bot non puo' parlare per al
momento".

**RISOLTO lo stesso giorno disattivando Express mode** (trigger -> Express mode (Preview) ->
"Enable faster flow execution" OFF, poi Save draft + reload + Publish).

Misure sulla stessa ricerca ampia "articoli per il laboratorio":
- con Express mode: **TimedOut a 02:00**
- senza Express mode: **00:51, Succeeded**, 12 articoli disponibili restituiti.

Express mode non stava accelerando nulla: il caso di regressione girava in 01:09 con Express
e la ricerca ampia in 00:51 senza. **Non riattivarlo.** Vedi [[rina-suite-regressione]]: i
tempi sono cambiati, la suite va rilanciata.
