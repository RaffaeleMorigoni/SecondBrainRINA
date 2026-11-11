---
name: rina-ricerca-parole-italiano-iniziaper
description: "La ricerca per parole di Search Items gira su ProductSearchName (sempre italiano) e con 'inizia con', non 'contiene': spiega i casi in cui il flusso parte e non trova"
metadata:
  node_type: memory
  type: project
---

**Letto sul flusso il 29/10/2026.** `Lista_prodotti_1..4`, entita' `ProductsV2`:

```
AVY0587_PurchaseNature eq ...'Hybrid' and ( ProductSearchName eq 'parola*' or ProductNumber eq 'parola*' ... )
```

Due limiti che si sommano:

1. **`ProductSearchName` e' sempre in italiano.** Le traduzioni esistono
   ([[rina-traduzioni-prodotto-esistono]]) ma la catena del 16/10 agisce **a valle**: traduce la
   descrizione dei codici **gia' trovati**. Traduciamo l'uscita, non l'ingresso. Una domanda in
   inglese produce keyword inglesi e **zero** risultati.
2. **Il confronto e' "inizia con"**, il wildcard sta solo in coda. Per trovare un codice bisogna
   indovinare **la prima parola** della sua descrizione italiana.

Prova sulla stessa query, cross-company:

| `ProductSearchName eq '<parola>*'` | risultati |
|---|---|
| `manutenzioni` | 5, fra cui SE108002.0001 |
| `meccanici` | **0** |
| `maintenance` / `mechanical` / `height` | 0 |

`meccanici` a zero e' il dato che chiude il caso: *"Manutenzioni impianti meccanici in quota"* non
**inizia** con "meccanici".

**Attenzione a come si leggono i test "riusciti"**: una domanda che contiene la descrizione
copiata dall'anagrafica contiene anche la sua prima parola. Non prova che la ricerca funzioni.

**Correzione piu' economica**: far generare al modello le parole chiave **in italiano**, qualunque
sia la lingua della domanda — una riga nelle istruzioni del figlio, nessun impatto sui tempi
([[rina-loop-contabilizzazione-servizi]] no; il timeout e' 2494). La descrizione esce comunque
tradotta.

`contains()` **non e' supportato** su `ProductsV2`: *"The type 'System.String' for the query
operator is not Queryable"*. Il "inizia con" non si sostituisce con una parola.

Il secondo canale (ricerca per categoria, [[rina-tassonomia-nuova-senza-descrizioni]]) copre una
parte dei casi: per questo il difetto non si manifesta sempre.

Dettaglio e schermate in `plans/20261029_ricerca_in_inglese_non_trova.md`.

## 29/10 - rettifica: convivono DUE forme, e quella con `*` in testa e' il collo di bottiglia

Letto sull'input di una run reale (`Lista_prodotti_1`, domanda `item number EX102002.0001`):

```
ProductSearchName eq '*EX102002.0001*' or ProductNumber eq 'EX102002.0001*'
```

**Il wildcard sta anche in testa**: quello e' un *contiene*, non un *inizia con*. Le due forme
convivono nello stesso `$filter` a seconda dell'indice della parola e della sua lunghezza —
prima di modificarlo va riletto per intero, non a campione.

`'*...*'` su un campo non indicizzato costringe alla **scansione piena**: in quella run
`Lista prodotti 1`, `2` e `4` **non sono mai tornate** (1 retry su Lista prodotti 1), mentre
`Lista prodotti 3` ha chiuso in 21s. Flusso in `TimedOut` a `00:02:00`, tetto di **Express mode**
([[rina-express-mode-attivo]]).

**Caso facile da chiudere**: se la parola ha la forma di un codice MMD, cercarla **dentro le
descrizioni non serve** — basta `ProductNumber`. Il contains e' lavoro buttato che costa il
timeout.

## 29/10 ore 15:01 — FIX APPLICATO sul ramo codice: 02:00 TimedOut → 00:00:44 Succeeded

Nelle quattro `Lista prodotti`, campo *Filter Query*: **se la parola contiene un punto, si
interroga solo `ProductNumber`**. Sparisce il `ProductSearchName eq '*codice*'`.

Misure sulla stessa domanda (`item number EX102002.0001`, RSSE, riga a video):
run `02:27 PM` **00:02:00 TimedOut** → run `03:05 PM` **00:00:44 Succeeded**.
Sull'anagrafica la query nuova risponde in **1,1 secondi**.

Nella stessa riscrittura, due pulizie a costo zero:
- `#none#` → `ZZNONEZZ`, erano gli ultimi quattro campi col debito del 05/10
  ([[rina-sentinella-none-filtri]]: ora chiuso);
- `decodeUriComponent('%27')` → **apostrofo raddoppiato**, stesso risultato e espressione da ~900
  a ~710 caratteri. **E' il modo per far stare un'espressione lunga nel campo**: la digitazione
  passa, mentre con decodeUriComponent andava in timeout. Da riusare.

**La ricerca per descrizione non e' toccata**: wildcard in testa ancora li', e' il 2494.

Punto di ripristino: `plans/backup/20261029_ROLLBACK_lista_prodotti_filtro_codice.md`.

## 29/10 ore 15:40 — RISOLTO: era una riga FALSA nelle istruzioni del figlio

Nel blocco `ITEM SEARCH AND ITEM INFORMATION CARD` c'era scritto:
*"**Item descriptions are in English** while procurement category names are in Italian and
English, so add English equivalents only in the keyword slots left over..."*

**Falso**: la ricerca gira su `ProductSearchName`, che e' italiano. Il modello non disobbediva,
**faceva quello che c'era scritto**: con domanda inglese spendeva keyword inglesi, non trovava
nulla e tornava un aggancio casuale.

Sostituita con *"The item descriptions the search matches are in Italian, so the keywords must
always be Italian words, whatever language the user writes in... This is about the search only:
keep answering the user in their own language."* — regola **generica**, nessuna lista di lingue.

**Servivano DUE sostituzioni**: poco sopra l'esempio diceva ancora *"laboratorio and only then an
English word such as equipment"*. Una riga che contraddice viene seguita al posto della regola
(gia' visto in [[rina-search-items-etichette-disponibilita]]).

Istruzioni del figlio da 29.327 a **29.447** caratteri.

**Esito su RSSE**, stessa domanda inglese di Emanuela:
prima `PS301003.0034 Business Finder - management systems` →
dopo **`SE108002.0001 High-altitude mechanical systems maintenance`**, con la descrizione
**in inglese**: la catena traduzioni funziona appena la ricerca trova qualcosa.
Resta la prosa italiana sotto il codice, che e' il tema noto della prosa.

**La lezione**: prima di correggere il comportamento, controllare che le istruzioni non
affermino qualcosa di **falso sui dati**. Stessa famiglia di [[rina-permesso-dati-pagina]]:
il modello eseguiva, non sbagliava.

Ripristino: `plans/backup/20261029_ROLLBACK_child_keyword_italiane.md`.
