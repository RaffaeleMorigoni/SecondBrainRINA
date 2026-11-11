# ROLLBACK — blocco PROCUREMENT RESULTS del padre (10/11/2026)

Orchestratore `Copilot for finance and operations apps`. Istruzioni a **7.973 / 8.000** prima
della modifica (27 caratteri liberi).

## Testo ORIGINALE del blocco

```
PROCUREMENT RESULTS: that agent never writes to the user. Your reply for the turn is its resolvedResult, sent once, with the MMD label, all items with code and description, and no version of your own before or after it. Write it in the user's language, never reformat it or drop its MMD label.sent once and copied as it is, with the MMD label, Each item: code plus description, never field: value pairs. The prefix must be exactly MMD | : if it differs, fix it to MMD | .
```

Da notare: **contiene un frammento spurio**, `sent once and copied as it is, with the MMD label, `
attaccato senza spazio a `label.`. E' il residuo della modifica del 29/10
(`backup/20261029_ROLLBACK_padre_copied_as_it_is.md`), rimasto dopo il ripristino. Non e' una
frase, e' un pezzo di testo orfano.

## Perche' si interviene

Il caso **"Supporto tecnico onsite"** e' stato analizzato in Power Automate (run `Nov 10, 15:17`):

- keyword estratte: `supporto`, `tecnico`, `onsite`;
- `Lista prodotti 1` (supporto) ha restituito `PS201012.0004`;
- `Lista prodotti 2` (tecnico) ha restituito `PS652005.0001`, `PS101003.0007`, `PS101003.0017`.

**Tutti e quattro i codici in risposta vengono dalla ricerca: l'agente non ha inventato niente.**
Ha fatto quello che l'istruzione gli dice — *"all items with code and description"* — cioe'
elencarli tutti, anche `TECNICO MANUTENTORE ESTINTORI` e `Monitoraggio geotecnico`
(che matcha perche' contiene "tecnico" dentro "geotecnico").

**Quindi la causa non e' l'invenzione, e' il permesso di elencare tutto.** Quinto
permesso-che-contraddice della serie.

## Testo NUOVO

```
PROCUREMENT RESULTS: that agent never writes to the user. Your reply for the turn is its resolvedResult, sent once, with the MMD label, only the items that answer the request, with code and description, and no version of your own before or after it. If none of them fits, say so and list none. Write it in the user's language, never reformat it or drop its MMD label. Each item: code plus description, never field: value pairs. The prefix must be exactly MMD | : if it differs, fix it to MMD | .
```

Tre cambi:
1. `all items` -> **`only the items that answer the request`**;
2. aggiunta **`If none of them fits, say so and list none.`**;
3. **rimosso il frammento spurio** - e' lo spazio che finanzia i primi due.

Conteggio: da 7.973 a **7.966 / 8.000** (sette caratteri in meno di prima).

---

## ESITO — pubblicato alle 16:43, funziona

**Il caso onsite e' risolto:**

> *MMD | Nessun articolo trovato corrisponde in modo sufficientemente preciso a
> «Supporto tecnico onsite».*

E' letteralmente il comportamento che Dafne e Alba si aspettavano.

**Non regressione verificata** su *"Devo acquistare un servizio di manutenzione per le
attrezzature di laboratorio"*:

> *MMD | SE153005.0001 - Manutenzione altre apparecchiature specialistiche di laboratorio*

Codice pertinente, quindi l'agente **non e' diventato cieco**.

## Un effetto collaterale da sorvegliare

La stessa domanda il **19/10 restituiva sei codici** (`MA101036`, `SE152002`, `SE303002`,
`SE153002`, `SE153005`, `SE651501`); **oggi ne restituisce uno solo**. Il codice dato e'
centrato, ma **la rosa di alternative e' sparita**.

Per una richiesta generica una rosa e' spesso piu' utile di un codice secco: l'utente sceglie.
**Da sorvegliare**: se i tester iniziano a dire *"mi da' un solo codice e non e' quello che
volevo"*, la causa e' questa modifica e va calibrata (es. *"only the items that answer the
request"* -> *"the items that answer the request, from the closest down"*).

Conteggio finale: **7.989 / 8.000**, 11 caratteri liberi.

Prove: `prove_uat/20261110_DOPO_onsite_nessun_articolo.png`,
`prove_uat/20261110_DOPO_non_regressione_manutenzione.png`.
