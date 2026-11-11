# ROLLBACK — istruzioni del figlio, parole chiave in italiano (29/10/2026)

Agente **RINA Procurement & MMD Assistant** (`b4a1f29d-c9f8-4c8d-889c-f1906ea65b5f`),
blocco `ITEM SEARCH AND ITEM INFORMATION CARD`. Campo *Instructions*, **29.327 caratteri** prima
della modifica. Una sola occorrenza, posizione 13.200.

## Frase ORIGINALE (per ripristinare, rimettere esattamente questa)

```
Item descriptions are in English while procurement category names are in Italian and English, so add English equivalents only in the keyword slots left over once every part of the request has one.
```

## Frase NUOVA

```
The item descriptions the search matches are in Italian, so the keywords must always be Italian words, whatever language the user writes in: translate the need into Italian before calling the tool, and never spend a keyword slot on an English equivalent. This is about the search only: keep answering the user in their own language.
```

## Perche'

La frase originale e' **falsa**. Verificato sull'anagrafica il 29/10: la ricerca per parole gira
su `ProductSearchName`, che e' **italiano** (`ProductName` su `ProductsV2` e' vuoto). Le
descrizioni tradotte esistono in `ProductTranslations` ma sono usate **a valle**, per scrivere la
risposta, mai per cercare.

Dicendo al modello che le descrizioni sono inglesi, le istruzioni lo spingevano a spendere le
keyword in inglese: con una domanda inglese la ricerca non trovava nulla e restituiva agganci
casuali. Riprodotto il 29/10 su RSSE:

- it — *"Ciao devo fare una RDA per questa attivita Manutenzioni impianti meccanici in quota..."*
  → `SE108002.0001`, **corretto**
- en — *"...maintenance work on mechanical systems at height. Which material should I use?"*
  → `PS301003.0034 Business Finder - management systems`, **fuori bersaglio**

Stesso difetto segnalato su FOFI (`ME301007.0001 Network device accessories`).
