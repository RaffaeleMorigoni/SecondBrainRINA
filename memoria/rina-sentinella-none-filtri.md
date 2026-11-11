---
name: rina-sentinella-none-filtri
description: Il sentinella #none# nei filtri OData di RINA - Search Items rompeva le ricerche da codice vecchio; sostituito con ZZNONEZZ
metadata:
  type: project
---

Nel flusso `RINA - Search Items` i rami "nessun risultato" usavano il valore sentinella `'#none#'`
(un codice che non esiste, per non lasciare il filtro vuoto). Il carattere `#` rompe la query:
Dynamics riceve la stringa **troncata all'`#`** e risponde `unterminated string literal at position N`.
Era la causa dei fallimenti sui casi che partono da un codice della vecchia nomenclatura
(`Npprom141`, `Npinfo521`), non un problema di infrastruttura come sembrava dal `BadGateway`
con cui Copilot Studio avvolge l'errore.

Il guasto si manifesta solo sull'entità **ReleasedProductsV2**: su `ProductsV2` lo stesso
`(ProductNumber eq '#none#')` passa. Per questo prima sembrava che il `#` non c'entrasse.
La prova decisiva: corretta `Lista_articoli`, il guasto si e' spostato su `Lista_altre_LE`,
l'azione successiva che aveva ancora `#none#`.

**Corretti il 05/10/2026 con `ZZNONEZZ`** (7 campi, pubblicati e verificati):
`Componi_clausole`, `Componi_clausole_prodotti`, `Lista_articoli`, `Lista_altre_LE`,
`Lista_categorie`, `Lista_assegnazioni`, `Lista_nature`.

**Debito residuo:** i quattro `Lista_prodotti_1..4` hanno ancora `#none#`. Sono su `ProductsV2`
quindi non rompono, ma l'espressione e' di ~900 caratteri e la digitazione nel campo va in timeout:
vanno riscritti con piu' calma. Guardarli per primi se ricompare un `unterminated string literal`.

**Why:** la stessa famiglia di difetti era gia' nota per la `&` (vedi [[copilot-studio-browser-quirks]]):
mai mettere caratteri speciali dentro i literal di un `$filter`.
**How to apply:** qualsiasi nuovo sentinella o valore letterale nei filtri deve essere
alfanumerico puro. Suite di riferimento: [[rina-suite-regressione]].

**09/10/2026 - i codici legacy li consegnava il flusso, non il modello.** Riga 31 del registro UAT:
l'agente mostrava `Npcons410`, `Npeste600`, `Npinfo510`. Due giri di divieti nelle istruzioni non
l'hanno chiuso. La causa sta in `RINA - Search Items`, azione **`Seleziona categorie`**, che
costruiva ogni voce della risposta come:

```
@concat(item()?['EcoResCategory_Name'], ' - ', item()?['FriendlyName'])
```

cioe' `Npcons410 - Consulenza per certificazioni (es. ISO 9001/...)`, **la stringa comparsa parola
per parola nella risposta**. Nessuna istruzione poteva vincere: il dato arrivava dallo strumento e
il modello lo trattava come legittimo.

**Fix pubblicato:** `select` ridotto a `@item()?['FriendlyName']`. Resta la descrizione leggibile,
sparisce il codice.

**Il punto da non sbagliare: ci sono DUE rami che partono da `Lista categorie`.**
- ramo **interno**, `Seleziona nomi categoria` → `Componi clausole assegn` → `Lista assegnazioni`:
  qui `EcoResCategory_Name` e' la **chiave** con cui si trovano gli articoli della categoria.
  Filtrare o togliere il campo qui romperebbe la ricerca per categoria, che e' l'unica che
  funziona finche' la tassonomia nuova resta muta (vedi [[rina-tassonomia-nuova-senza-descrizioni]]).
- ramo **di risposta**, `Filtra categorie it` → `Seleziona categorie` → `Rispondi all agente`:
  e' l'unico da correggere.

**Due trappole dell'editor incontrate:**
1. togliendo il chip dell'espressione, il campo Map passa in modalita' testo e il designer segnala
   **"Invalid parameters" / "Enter a valid JSON"** anche quando il valore e' corretto. E'
   validazione stantia: `Save draft`, **ricaricare la pagina**, e l'avviso sparisce. Il codice in
   Code view e' la fonte attendibile, non il badge.
2. vale sempre [[power-automate-espressioni-non-si-salvano]]: scrivere a mano nel campo con la `@`.

Punto di ripristino: `scratchpad/ROLLBACK_search_items_seleziona_categorie.md`.

**Verificato in F&O il 09/10/2026, utente C_110912 su RSSE:**
- "MMD code per una prestazione professionale in ambito 9001" → due sezioni, solo
  `PS101002.0002 - ISO 9001`, **nessun codice in vecchia nomenclatura**. Prima usciva
  `Npcons410 — Consulenza per certificazioni ... come categoria di approvvigionamento`.
- controprova, "attrezzatura e materiali da laboratorio" → `ME201003.0001`, `ME201001.0001`,
  `MA101037.0001` e `SE707502.0001` fra i non rilasciati: **la ricerca per categoria regge**.

Due tentativi di chiudere il difetto dalle istruzioni erano falliti; una riga nel flusso l'ha
chiuso al primo colpo. Quando il dato sbagliato e' *costruito* da un'azione del flusso, cercarlo
nelle istruzioni e' tempo perso.
