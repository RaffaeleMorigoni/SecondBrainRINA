# Segnalazione Emanuela: la lingua dei campi dati — chiuso

30/10/2026. Emanuela: *"continuano ad esserci problemi con la lingua nelle risposte"*, con uno
scambio in francese su una RDA per certificazione ISO 9001.

## Cosa mostrava lo screenshot

La **prosa era francese e corretta**. A uscire in italiano erano i **campi che arrivano da
Dynamics**, con la traduzione francese appiccicata fra parentesi:

- `2 - D - Costo OPEX diretto [cout OPEX direct]`
- `Compte 411000 - Servizi produttivi di terzi [services de production de tiers]`

## La causa: non un bug, un ordine sbagliato

Nel blocco LANGUAGE del figlio `b4a1f29d` c'era questa frase:

> *Item descriptions, category names and accounting type descriptions come from Dynamics in their
> original language, **usually English**: always show that original text unchanged so the user can
> match it in F&O, and when the user writes in another language add a short translation in
> brackets right after it.*

**La premessa e' falsa**: le descrizioni arrivano **in italiano**, non in inglese. Partendo da li',
*"mostra l'originale invariato"* ordina di stampare l'italiano. L'agente stava obbedendo — lo
stesso schema del 2418 ([[rina-permesso-dati-pagina]]) e delle keyword.

E la ragione addotta — *"so the user can match it in F&O"* — non regge piu': le traduzioni
esistono in `ProductTranslations` per 7 lingue ([[rina-traduzioni-prodotto-esistono]]), quindi in
F&O un utente francese vede il francese.

## La correzione, in due passaggi

**Primo tentativo** (pubblicato 16:5x): detto che i campi arrivano in italiano e che vanno scritti
nella lingua dell'utente, *"adding the Italian original in brackets only when no translation is
available"*.

Esito: le **descrizioni prodotto** sono diventate francesi pulite, ma tipi di contabilizzazione e
nomi dei conti sono usciti in **italiano nudo**, senza nemmeno la parentesi di prima — **peggio**
del punto di partenza. Per quei campi Dynamics non ha traduzioni, e la mia regola diceva di
scrivere l'italiano proprio in quel caso. **L'avevo scritta al contrario.**

**Seconda versione** (pubblicata 17:13), coda della frase riscritta:

> *…and when Dynamics has no translation for a field, such as accounting type descriptions and
> account names, translate the Italian text yourself into the user's language and put the Italian
> original in brackets right after it.*

## Esito verificato in F&O, stessa domanda di Emanuela

```
PS101002.0002 — Activites de certification des systemes de management de la qualite
                — Service — categorie External — unite UC_DHI08.
2 - D - Cout OPEX direct [Costo OPEX diretto] — compte 411000 - Services productifs de tiers
        [Servizi produttivi di terzi] — categorie projet External.
SE355502.0001 — Organismes de certification — Service — categorie Accrfees — unite LUMPSUM,
        avec 1 - I - Cout OPEX indirect [Costo OPEX indiretto], compte 411102 - Frais d'accreditation
```

Francese in chiaro, italiano fra parentesi per il riscontro, **codici e nomi propri invariati**
(`External`, `Accrfees`, `LUMPSUM`, `UC_DHI08`).

Istruzioni del figlio da **29.447** a **29.550** caratteri.
Ripristino: `backup/20261030_ROLLBACK_figlio_riga_lingua.md`.

## Resta aperto su questo stesso caso

Non ho ancora verificato che `PS101002.0002` **venga dalla ricerca** e non dalla Knowledge:
e' la regola nuova di oggi ([[rina-codici-fuori-dalla-ricerca]]). Il fix sulla lingua e' chiuso,
quello sulla provenienza dei codici no.

Prove: `prove_uat/20261030_passaggio1_italiano_nudo.png`,
`prove_uat/20261030_DOPO_fix_lingua_francese_con_italiano_in_parentesi.png`.
