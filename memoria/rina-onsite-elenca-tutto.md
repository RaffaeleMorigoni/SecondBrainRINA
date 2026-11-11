---
name: rina-onsite-elenca-tutto
description: "Il difetto onsite non era invenzione: i codici venivano davvero dalla ricerca, e l'istruzione diceva di elencarli TUTTI. Risolto con 'only the items that answer the request'"
metadata:
  type: project
---

**10/11/2026 — diagnosi, e rettifica di quella del 30/10.**

Caso *"Supporto tecnico onsite"*: l'agente proponeva quattro codici assurdi
(`TECNICO MANUTENTORE ESTINTORI`, `Monitoraggio geotecnico`…) a fronte di **zero articoli che
contengono "onsite"**.

**Letta la run in Power Automate, tutti e quattro venivano dalla ricerca:**
- keyword: `supporto`, `tecnico`, `onsite`;
- `Lista prodotti 1` (supporto) -> `PS201012.0004` (*Laser scanning per supporto alla progettazione*);
- `Lista prodotti 2` (tecnico) -> `PS652005.0001`, `PS101003.0007`, `PS101003.0017`.

`Monitoraggio geo**tecnico**` matcha perche' il `*tecnico*` prende anche le parole composte.

**Quindi l'agente non inventava niente: obbediva.** Il blocco PROCUREMENT RESULTS diceva
*"…with the MMD label, **all items** with code and description"*. **Quinto
permesso-che-contraddice** della serie ([[rina-permesso-dati-pagina]],
[[rina-lingua-campi-dynamics]], [[rina-blocco-language-ambito]],
[[rina-variabili-lingua-dal-locale]]).

**Rettifica a [[rina-codici-fuori-dalla-ricerca]]**: quella diagnosi (codici non presenti negli
output) vale per il caso francese del 30/10, **non per onsite**. Sono due fenomeni distinti e
vanno tenuti separati.

**Fix pubblicato (10/11, 16:43)**: `all items` -> **`only the items that answer the request`**,
piu' **`If none fits, say so and list none.`**. Lo spazio e' stato finanziato **rimuovendo un
frammento spurio** rimasto dal ripristino del 29/10 (`sent once and copied as it is, with the MMD
label, ` incollato senza spazio dopo `label.`): **le istruzioni contenevano testo orfano**, da
cercare anche altrove.

**Esito**: *"Nessun articolo trovato corrisponde in modo sufficientemente preciso"* - come atteso.
Non regressione OK (`SE153005.0001` su manutenzione laboratorio).

**Effetto collaterale da sorvegliare**: la stessa domanda il 19/10 dava **sei** codici, oggi
**uno**. La rosa di alternative si e' ristretta. Se i tester lamentano *"un solo codice e non e'
quello giusto"*, la causa e' questa e va calibrata.

Istruzioni padre: **7.989 / 8.000**. Ripristino in
`plans/backup/20261110_ROLLBACK_padre_procurement_results.md`.
