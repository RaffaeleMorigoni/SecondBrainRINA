---
name: rina-timeout-search-items
description: il timeout client su Search Items scatta intorno ai 60 secondi, non ai 2 minuti
metadata:
  type: project
---

**Misurato il 19/10/2026.** `FlowActionTimedOut` / 504 Gateway Timeout su `Rispondi all agente`.

La soglia del client **non e' 2 minuti**: e' intorno al **minuto**. Prove dello stesso giorno:

| run | durata | esito |
|---|---|---|
| 11:49 | 33s | Succeeded |
| 12:12 | 47s | Succeeded |
| 12:12 | 1m 43s | **Failed** |
| 10:10 | 4m 07s | **Failed** |

## Cosa e' gia' stato fatto (pubblicato il 19/10)

Condizione `Serve la societa` in cima allo scope `Cerca articoli`: `Lista utente` e
`Lista system user` girano solo quando `CompanyId` arriva vuoto. Da F&O arriva sempre, quindi
vengono saltate: **0,1s invece di 2m 13s**. Vedi [[power-automate-espressioni-non-si-salvano]]
per il trucco della condizione stringa-contro-stringa.

## Cosa resta, con i numeri della run fallita delle 12:12 (scope 1m 45s)

| azione | tempo |
|---|---|
| Serve la societa | 0,1s |
| Lista prodotti 1-4 (parallele) | ~30s (il massimo delle quattro) |
| Lista categorie / Lista assegnazioni | <1s |
| Lista articoli | 9s |
| **Lista nature** | **26s** |
| **Lista altre LE** (parallela con Lista traduzioni 30s) | **37s** |

Somma del percorso critico ~103s. I tre blocchi pesanti sono **sequenziali**: Lista prodotti ->
Lista articoli -> Lista nature -> Lista altre LE. Finche' restano in fila, nelle ore in cui UAT
e' lento il flusso sfora comunque.

**Come usarlo**: non dire che il timeout e' risolto. E' risolto il contributo delle due query
sulla societa'. Se si ripresenta, guardare quei tre blocchi, non le azioni sulle traduzioni
(Lista traduzioni e' in parallelo e non e' mai sul percorso critico).

## 19/10/2026 pomeriggio - due strade provate e chiuse

**Fusione di `Lista articoli` e `Lista altre LE` in una query sola: SCARTATA.**
In UAT ci sono **9 societa'** (`rent prid patr foee fofi fouk fous rsse rtur`), quindi senza il
vincolo di societa' ogni codice rende 9 righe. Misurato su 150 codici, 3 giri: la query fusa
costa 4,1-5,7 s contro i 2,3-3,5 s della somma delle due attuali, e tocca il tetto di `$top`
(2000 righe esatte) perdendo dati. Con 40 codici il risultato era opposto (574 ms contro 2138):
**la convenienza dipende dal numero di codici e con numeri realistici non c'e'**. Nessuna
modifica fatta.

**Il tempo non e' nelle query, e' nel connettore.** Le stesse query via OData diretto:
Lista articoli ~0,4-1,4 s contro i **9 s** come azione del flusso; Lista altre LE ~1,8-2,5 s
contro **37 s**. Un ordine di grandezza. Ottimizzare filtri, `$top` o `$select` non sposta
niente: conta solo **quante chiamate al connettore** fa il flusso.

**Express mode: non attivabile, serve una licenza.** E' l'interruttore che agisce proprio su
quell'overhead, e il designer lo suggerisce da giorni. Attivando il toggle, Power Automate
risponde: «Assign a user or capacity license to this flow to use premium features and dedicated
capacity.» Il flusso e' su `Plan = The user who runs the flow`; serve una **Process license** o
capacita' dedicata. Il salvataggio viene accettato ma lo stato torna `Disabled`.

**Come usarlo**: e' una **decisione commerciale di RINA**, da portare al cliente. E' il modo piu'
efficace e meno invasivo di chiudere il timeout, perche' agisce sulla causa vera senza toccare
la logica. Non spendere altro tempo a ottimizzare le query.
