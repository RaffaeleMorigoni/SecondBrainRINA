---
name: rina-mcp-dynamics-solo-nel-test
description: "Il server MCP Dynamics e' in sola lettura e non connesso: funziona nel pannello di test ma non in F&O, ed e' un motivo in piu' per non fidarsi del test"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T14:43:13.668Z
---

Strumento **`MCP Dynamics 365 ERP Procurement & MMD`**
(`b5264188-3d8d-4e2e-98f2-a32fb85514c2`). E' il **Dynamics 365 ERP MCP di Microsoft**: accesso
generico a F&O via OData/SQL. **Configurato da noi (HUDi)**, non da Chiara: il suo nome nella
colonna *Last modified* dice solo chi ha toccato per ultimo la riga. Attenzione a non dedurre la
paternita' da quella colonna - stesso errore di [[rina-task-descrizione-non-registro]].
**Available to:** `RINA Procurement & MMD Assistant`. **Connection: Not connected.**

**E' in sola lettura**, verificato tool per tool il 21/10:

- **ON**: `data_find_entities_sql`, `data_find_entity_type`, `data_get_entity_metadata`
- **OFF**: `api_find_actions`, `api_invoke_action`, `data_create_entities`,
  `data_update_entities`, `data_delete_entities`, tutti i `form_*`

Niente scritture, niente pilotaggio dei form. Ma `data_find_entities_sql` interroga **qualunque
entita'** bypassando i flussi: niente `ProductTranslations`, niente filtro di rilascio per
societa', niente formato concordato.

## Quando viene scelto: solo nel pannello di test

| ambiente | esito |
|---|---|
| pannello di test Copilot Studio | **invocato** (log: "The agent invoked the MCP server"), risposte con descrizioni inglesi e codici non filtrati |
| **F&O** | **mai usato**: due domande diverse, entrambe hanno generato run di `RINA - Search Items` |

La causa e' `Connection: Not connected`: nel test la connessione usa le credenziali di chi edita,
in esercizio no.

## Cosa portarsi dietro

1. **L'MCP non spiega il 2418 in F&O.** Corregge [[rina-knowledge-compete-con-la-ricerca]]: in
   F&O l'MCP non e' raggiungibile, quindi li' la fonte era un'altra.
2. **Un motivo in piu' per non fidarsi del pannello di test**, oltre a quelli gia' noti
   ([[rina-formato-fo-non-governabile]]): nel test l'agente ha una via d'accesso ai dati che in
   esercizio non ha. **I test di comportamento si fanno in F&O.**
3. **Se quella connessione viene stabilita**, in esercizio comparira' una seconda fonte che
   scavalca i flussi, con i problemi di lingua e di filtro societa' gia' visti nel test.
   **La decisione e' nostra**, lo strumento l'abbiamo messo noi: va presa prima che la connessione
   venga attivata.


## RETTIFICA del 28/10: non e' innocuo, BLOCCA gli utenti

Avevo scritto che in esercizio l'MCP era innocuo perche' `Not connected`, e che il danno era solo
nel pannello di test. **Sbagliato.**

Nel pannello **Share** dell'agente, l'avviso *"Your agent runs tools that use the author's
credentials"* elenca **6 strumenti: tutti e soli i flussi Power Automate** (Check Vendor,
Get Item Accounting Types x2, Search Items x2, Category Ledger Accounts). **I due MCP non ci
sono.**

Quindi l'MCP **non eredita le credenziali dell'autore**: pretende la connessione del singolo
utente, che non c'e'. Quando l'orchestratore lo sceglie, l'utente riceve la card
**"Open connection manager to verify your credentials"** e **non puo' proseguire**.

Successo davvero: **28/10 ore 8:06**, un utente RINA chiede *"mmd what is the product ?"*, riceve
quella card tre volte, nessuna risposta. La sessione risulta **"Resolved"**.

**Non connesso non vuol dire ignorato.** Finche' i due MCP restano attivi, qualunque utente puo'
sbatterci contro. Sono in sola lettura e non servono a nessuna funzione: **da disattivare o
connettere**, e la decisione e' nostra perche' li abbiamo messi noi.
