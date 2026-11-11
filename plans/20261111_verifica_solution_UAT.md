# Verifica in ambiente della solution — UAT

**Data:** 11/11/2026 — sola lettura, nessuna modifica eseguita
**Ambiente:** `rina-uat-clm` (`a485debc-a3ef-e411-8613-fcec887d964f`)

---

## La solution

| | |
|---|---|
| Nome | **Dynamics 365 F&O Procurement Expert** (`dynamics365foprocurementexpert`) |
| Id | `fe0a1b50-947e-f111-ab0e-70a8a537aa21` |
| Package type | **Unmanaged** (corretto per l'ambiente sorgente) |
| Publisher | **HUDi Digital Humanism** |
| Versione | 1.0.0.2 |
| Creata | 13/07/2026 |
| Export già eseguiti | 1.0.0.1 e 1.0.0.2, entrambi riusciti |
| Pipeline | **nessuna configurata** |

Buona notizia di fondo: la solution custom **esiste già**, con publisher nostro, ed è
unmanaged. Il lavoro di impianto è fatto.

## Contenuto — 660 oggetti

| categoria | n. |
|---|---|
| Agent components | 648 |
| Agents | 2 |
| Cloud flows | 6 |
| Connection references | 4 |
| Apps / Cards / Data workspaces / Tables | 0 |
| **Environment variables** | **0** |

648 + 2 + 6 + 4 = 660: nella solution **non c'è nessuna environment variable**.

---

## Rilievo 1 — L'orchestratore managed è dentro la solution

**Agents (2):**

| Display name | Managed | Customized | Ultima modifica | Owner |
|---|---|---|---|---|
| **Copilot for finance and operations apps** | **Yes** | **Yes** | 17 ore fa | DYN_365_ADMIN |
| RINA Procurement & MMD Assistant | No | Yes | 1 settimana fa | Chiara DAMBROSIO |

**Solution layers dell'orchestratore** (`679d8694-1250-4780-946e-6db1c2109ae1`):

| Order | Solution | Publisher | Layer status |
|---|---|---|---|
| 2 | **Unmanaged layer** | Default Publisher for operations-rina-uat-clm | **Active** |
| 1 | `msdyn_FnOCopilot` | Dynamics 365 | (base managed) |

È esattamente il caso previsto: componente managed di Microsoft con le nostre
customizzazioni in un **unmanaged layer Active** sopra. Confermato anche il
"17 ore fa", coerente con le modifiche di ieri.

**Per contrasto, i layer del figlio** (`b4a1f29d-c9f8-4c8d-889c-f1906ea65b5f`):

| Order | Solution | Layer status |
|---|---|---|
| 1 | Unmanaged layer | Active |

Un solo layer, nessun managed sotto: componente interamente nostro. Questo è il caso
pulito, e si deploya senza problemi.

### Che cosa resta da accertare

Non è dimostrato che l'export **catturi** le istruzioni del padre. Dataverse in generale
sa trasportare l'estensione di un componente managed, ma per gli agenti Copilot Studio la
documentazione descrive solo come *rimuovere* un unmanaged layer da un agente managed,
mai come esportarlo.

**Il modo definitivo per saperlo:** aprire lo zip dell'export 1.0.0.2 (già prodotto) e
guardare se il bot `679d8694…` c'è dentro con le sue istruzioni, o solo come riferimento.
Costa cinque minuti e toglie l'unica incertezza rimasta nel piano di rilascio.

---

## Rilievo 2 — Nella solution ci sono componenti di Microsoft, non nostri

**Connection references (4)**, tutte unmanaged, tutte in stato *Off*:

| nome | owner |
|---|---|
| `cr824_RINAProcurementMMDAssistant.shared_dynamicsax.shared-dynamicsax-5526499a…` | Chiara |
| **Fin & Ops Apps (Dynamics 365)** — `new_shareddynamicsax_e0108` | Raffaele |
| `msdyn_fnocopilot.shared_dynamicsax.02719d14e9fb4622a954c871e1617349` | Chiara |
| `msdyn_fnocopilot.shared_dynamicsax.shared-dynamicsax-5526499a…` | Chiara |

**Due su quattro hanno prefisso `msdyn_fnocopilot`**: appartengono all'app managed di
Microsoft, non a noi. Sono probabilmente finite dentro con un *Add required objects*.

Perché è un rischio: importando in produzione una solution che le contiene, si va a
toccare la configurazione di connessione dell'app Microsoft in prod. Da valutare se
possono uscire dalla solution — e se non possono, da sapere prima dell'import, non dopo.

**Le 2 environment variable da aggiornare** sono anch'esse estranee all'agente:

- *System Optimization Dashboard API config* (vuota)
- *QEA Fabric MCP Server URL* (vuota)

Nulla a che vedere con il Procurement Assistant. **Non le ho toccate**, ho chiuso il
pannello con Cancel. Vanno capite prima del rilascio: se restano vuote e la solution le
pretende, l'import in prod può chiederne il valore o fallire.

---

## Rilievo 3 — I flussi sono tutti dentro, e tutti accesi

**Cloud flows (6)**, tutti unmanaged, tutti *On*:

| flusso | owner |
|---|---|
| RINA - Check Vendor | Raffaele |
| RINA - Create RDA Draft | Chiara |
| RINA - Get Category Accounts | Raffaele |
| RINA - Get Item Accounting Types | Raffaele |
| RINA - Item Purchase History | Raffaele |
| RINA - Search Items | Raffaele |

Questo chiude anche la vecchia questione dei "flussi scollegati": i sei ci sono tutti e
sono nella solution.

---

## Rilievo 4 — La pipeline non esiste ancora, e la produzione va licenziata

Nessuna pipeline configurata. L'avviso in pagina è il vincolo da portare a RINA:

> *Target Environments in pipelines must be enabled as Managed Environments.*

**La produzione dovrà essere un Managed Environment.** È una questione di licenza
(Power Platform premium), non di configurazione: va verificata con chi gestisce il tenant
prima di promettere una data di rilascio.

---

## Cosa farei adesso, in ordine

1. **Ispezionare lo zip 1.0.0.2** per sapere se le istruzioni del padre viaggiano. È la
   domanda che decide se il piano ha una parte manuale o due.
2. **Snapshot testuale** delle istruzioni del padre, della descrizione del connected
   agent e delle descrizioni degli strumenti, versionato qui nel repo. Serve comunque,
   qualunque sia la risposta al punto 1: è l'unico rollback di quella parte.
3. **Capire le due env var estranee** e le due connection reference `msdyn_fnocopilot`:
   se possono uscire dalla solution, la solution diventa pulita e il rischio di toccare
   roba di Microsoft in prod sparisce.
4. **Verificare che la produzione sia Managed Environment**, o farla abilitare.
5. Solo dopo: creare la pipeline e fissare la data.
