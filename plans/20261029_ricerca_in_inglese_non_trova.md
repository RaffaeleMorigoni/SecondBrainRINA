# La ricerca in inglese non trova quello che trova in italiano (FOFI)

Segnalazione del 29/10/2026, due schermate su **FOFI**. Verificato lo stesso giorno sul flusso e
sull'anagrafica.

| | domanda | risposta |
|---|---|---|
| it | "...questa attivita Manutenzioni impianti meccanici in quota, quale materiale devo usare?" | **SE108002.0001** Manutenzioni impianti meccanici in quota — corretta |
| en | "...maintenance work on mechanical systems at height. Which material should I use?" | **ME301007.0001** Network device accessories, piu' "No relevant item is currently listed" |

---

## Non e' il 2418

La risposta inglese ha **la struttura del flusso** (*Available items in your company* / *to be
requested*): `RINA - Search Items` **e' stato invocato**, e' partito, ha girato e **non ha trovato**.
Il 2418 e' il caso opposto, flusso non invocato. Difetto diverso, da tenere separato.

---

## La causa, letta sul flusso

`Lista_prodotti_1..4`, entita' **`ProductsV2`**, `$filter`:

```
AVY0587_PurchaseNature eq ...ItemPurchaseNature'Hybrid' and (
  ProductSearchName eq 'parola*' or ProductNumber eq 'parola*' ... )
```

Due fatti, nessuno dei due ovvio:

1. **si cerca su `ProductSearchName`, che e' sempre italiano.** Le traduzioni esistono
   (`ProductTranslations`, en-gb di SE108002.0001 = *"High-altitude mechanical systems
   maintenance"*, che conterrebbe tutte e tre le parole della domanda inglese) ma la catena
   costruita il 16/10 agisce **a valle**: traduce la descrizione dei codici gia' trovati.
   **Traduciamo l'uscita, non l'ingresso.**
2. **il confronto e' "inizia con", non "contiene"** — il wildcard sta solo in coda.

### La prova, sulla stessa query del flusso

`ProductSearchName eq '<parola>*'` su ProductsV2, cross-company:

| parola | risultati |
|---|---|
| `manutenzioni` | **5**, fra cui SE108002.0001 |
| `meccanici` | **0** |
| `maintenance` | **0** |
| `mechanical` | **0** |
| `height` | **0** |

`meccanici` a zero e' il dato che spiega tutto: *"Manutenzioni impianti meccanici in quota"* non
**inizia** con "meccanici". Per trovare un codice bisogna indovinare **la prima parola della sua
descrizione italiana**.

La domanda italiana ha funzionato perche' conteneva la descrizione copiata esatta dall'anagrafica,
quindi anche la sua prima parola. Non e' una ricerca riuscita: e' una ricerca a cui era stata data
la risposta.

---

## Cosa farei, in ordine

**1. Parole chiave in italiano** (una riga nelle istruzioni del figlio). Le keyword le genera il
modello, non l'utente: basta dirgli di produrle in italiano qualunque sia la lingua della domanda.
Non tocca il flusso, non tocca i tempi, non rischia il timeout (2494). La descrizione esce comunque
tradotta, perche' quella parte gia' funziona.

**2. Il "inizia con" resta**, e vale anche in italiano. Sistemarlo e' piu' invasivo:
`contains()` non e' supportato su `ProductsV2` (`The type 'System.String' for the query operator is
not Queryable`), quindi non e' una sostituzione di una parola. Va studiato a parte — oggi il
secondo canale (ricerca per categoria) copre una parte dei casi, e per questo il difetto non si
vede sempre.

**3. Cercare dentro `ProductTranslations`** nella lingua dell'utente: piu' corretto in teoria,
caro in pratica — un'altra query sul percorso critico, su un'entita' gia' nota come lenta.

Prima pero' chiude la verifica a freddo del 2418. **Una modifica alla volta.**

---

## Due note per chi legge la segnalazione

1. I codici delle due schermate **non coincidono**: la risposta italiana da' `SE108002.0001`
   (impianti meccanici), la griglia mostra `SE108001.0001` (edili). Entrambi esistono in FOFI.
2. Nella griglia il **Product name e' gia' inglese** (*High-altitude building maintenance*): chi
   guarda la pagina vede l'inglese e conclude che il dato c'e'. Il flusso guarda l'altro campo.

Schermate: `prove_uat/20261029_FOFI_italiano_vs_inglese_stessa_domanda.png`,
`prove_uat/20261029_FOFI_released_products_SE_search_name_italiano.png`.

---

# RISOLTO — 29/10, pubblicato alle 15:40

## Prima: riprodotto su RSSE con le domande di Emanuela

| | domanda | risposta |
|---|---|---|
| it | "Ciao devo fare una RDA per questa attivita Manutenzioni impianti meccanici in quota..." | **SE108002.0001** — corretto |
| en | "Hi, I need to create a purchase requisition for maintenance work on mechanical systems at height..." | **PS301003.0034** Business Finder - management systems — fuori bersaglio |

Stesso difetto del caso FOFI (`ME301007.0001 Network device accessories`): non e' la societa',
e' la lingua della domanda.

## La causa vera: una riga sbagliata nelle istruzioni del figlio

Nel blocco `ITEM SEARCH AND ITEM INFORMATION CARD` c'era scritto:

> *Item descriptions are in English while procurement category names are in Italian and English,
> so add English equivalents only in the keyword slots left over once every part of the request
> has one.*

**E' falso.** La ricerca per parole gira su `ProductSearchName`, che e' **italiano**
(`ProductName` su `ProductsV2` e' vuoto). Le traduzioni esistono in `ProductTranslations` ma
servono a **scrivere** la risposta, non a cercare.

Le istruzioni dicevano al modello l'opposto della realta': con una domanda inglese spendeva le
keyword in inglese, la ricerca non trovava nulla e tornava un aggancio casuale.
**Non e' il modello che sbagliava: faceva quello che c'era scritto.**

## Due sostituzioni, nessuna aggiunta

1. la frase sopra → *"The item descriptions the search matches are in Italian, so the keywords
   must always be Italian words, whatever language the user writes in: translate the need into
   Italian before calling the tool, and never spend a keyword slot on an English equivalent.
   This is about the search only: keep answering the user in their own language."*
2. l'esempio poco sopra diceva ancora *"...laboratorio and only then an English word such as
   equipment"* → *"...laboratorio, strumenti, all of them Italian."* Una riga che contraddice
   viene seguita al posto della regola: andavano corrette **entrambe**.

Istruzioni da **29.327 a 29.447** caratteri. Ripristino:
`backup/20261029_ROLLBACK_child_keyword_italiane.md`.

## Dopo

Stessa domanda inglese, sessione nuova:

> **MMD | SE108002.0001 High-altitude mechanical systems maintenance**
> Per una richiesta di acquisto relativa a lavori di manutenzione su impianti meccanici eseguiti
> in quota, questo e' l'articolo disponibile nella societa' RSSE...

Codice **corretto**, e la descrizione esce **in inglese** — la catena delle traduzioni fa il suo
lavoro appena la ricerca trova qualcosa.

**Resta la prosa italiana** sotto il codice: e' il tema noto della prosa, non della ricerca.
Prima non si vedeva perche' la risposta era sbagliata in partenza.

Prove: `prove_uat/20261029_RSSE_inglese_risultato_fuori_bersaglio.png` (prima),
`prove_uat/20261029_dopo_fix_keyword_italiane_inglese_ok.png` (dopo).

## Regressione

Il caso italiano rieseguito dopo il fix al filtro del flusso: `SE108002.0001`, corretto.
Nessuna regressione.
