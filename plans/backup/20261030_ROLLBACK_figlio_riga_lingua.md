# ROLLBACK — figlio `b4a1f29d`, frase sulla lingua delle descrizioni (30/10/2026)

Agente figlio *RINA Procurement & MMD Assistant* connesso a *Copilot for finance and operations
apps*. Istruzioni a **29.447** caratteri prima della modifica. Blocco LANGUAGE.

## Testo ORIGINALE (da rimettere per annullare)

```
Item descriptions, category names and accounting type descriptions come from Dynamics in their original language, usually English: always show that original text unchanged so the user can match it in F&O, and when the user writes in another language add a short translation in brackets right after it, for example "Purchase of Hdw - Pcs (CAPEX) [acquisto hardware PC]". Never replace the original text with the translation and never translate the item code.
```

## Testo NUOVO

```
Item descriptions, category names and accounting type descriptions come from Dynamics in Italian, and F&O shows each user the translation in their own language: write them in the user's language, taking the translation the search returned, and add the Italian original in brackets right after it only when no translation is available. Never translate the item code, product names or any proper name: keep those exactly as they are.
```

## Perche'

La premessa **"usually English" e' falsa**: le descrizioni arrivano **in italiano**. Partendo da
quella falsita', *"always show that original text unchanged"* ordina di stampare l'italiano, e
*"add a short translation in brackets"* produce il misto che Emanuela ha segnalato il 30/10:

- `2 - D - Costo OPEX diretto [cout OPEX direct]`
- `Compte 411000 - Servizi produttivi di terzi [services de production de tiers]`

La ragione originale — *"so the user can match it in F&O"* — **non regge piu'**: le traduzioni
esistono in `ProductTranslations` per 7 lingue, quindi in F&O un utente francese vede il francese.
Mostrargli l'italiano non aiuta il riscontro, lo ostacola.

Il divieto di tradurre codici e nomi propri **resta**, ed e' quello che tiene corretto
`PS301003.0024 - Business Finder - iso 9001`.
