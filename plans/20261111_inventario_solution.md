# Inventario della solution — c'è tutto?

**Data:** 11/11/2026 — sola lettura
**Solution:** Dynamics 365 F&O Procurement Expert (`fe0a1b50-947e-f111-ab0e-70a8a537aa21`)

---

## Risposta breve

**I flussi e il figlio ci sono tutti. Mancano le knowledge, e mancano le
environment variable.** In più ci sono dentro cose che non sono nostre, e dei
duplicati da capire.

---

## C'è (verificato)

### I 6 flussi — completi, tutti unmanaged, tutti accesi

RINA - Check Vendor · Create RDA Draft · Get Category Accounts ·
Get Item Accounting Types · Item Purchase History · Search Items

### L'agente figlio e i suoi topic

`RINA Procurement & MMD Assistant`, un solo layer unmanaged.

Topic presenti (circa 40), fra cui quelli che contano davvero:

- **AUTONOMOUS Conversation & Agent Routing**
- **CHILD Conversation & Agent Routing**
- **Procurement & MMD**
- **Procurement & RDA Orchestration RINA**
- **RDA Creation Assistant**, **RDA Management**
- **RINA Out of Scope**, Fallback, Inizio conversazione, In caso di errore,
  Esegui escalation, e i topic di sistema (Salve, Grazie, A presto, Accedi…)
- i topic-strumento: RINA - Search Items, Check Vendor, Create RDA Draft,
  Get Category Accounts, Get Item Accounting Types, Item Purchase History
- tre topic **Fin & Ops Apps (Dynamics 365) - MCP Dynamics 365 ERP**

### Le 4 connection reference

Una ben nominata (`Fin & Ops Apps (Dynamics 365)`), una del figlio, e due di Microsoft
(vedi sotto).

---

## NON c'è

### 1. Knowledge: zero

Filtrando gli agent components per tipo **Knowledge** la solution restituisce
*"We didn't find anything to show here"*.

Gli Excel dei codici e la guida CAPEX/OPEX **non viaggiano con la solution**. Vanno
ricaricate a mano in produzione e portate a stato *Ready* (caricate non basta: devono
finire l'indicizzazione).

Era l'ipotesi della documentazione — ora è un fatto misurato.

### 2. Environment variable: zero

660 oggetti totali = 648 agent components + 2 agents + 6 flussi + 4 connection
reference. Non resta spazio: **nella solution non c'è nessuna environment variable**.

Da verificare se i sei flussi ne usino. Se sì, all'import in produzione mancano.

### 3. Skill: zero

Nessun componente di tipo Skill o Skill (V2). Gli strumenti sono rappresentati come
topic, non come skill — coerente con l'architettura, ma da sapere.

---

## Da capire prima del rilascio

### a) Nella solution ci sono componenti di Microsoft

Due connection reference con prefisso **`msdyn_fnocopilot`**, e l'agente managed
**Copilot for finance and operations apps** (`Managed = Yes`).

Importare in produzione una solution che li contiene significa andare a toccare
configurazione dell'app Microsoft in prod.

### b) Topic-strumento duplicati

| topic | occorrenze |
|---|---|
| RINA - Search Items | **5** |
| RINA - Get Item Accounting Types | **3** |
| Fin & Ops Apps (Dynamics 365) - MCP Dynamics 365 ERP | **3** |
| RINA - Check Vendor | 2 |
| RINA - Get Category Accounts | 2 |
| RINA - Create RDA Draft | 1 |
| RINA - Item Purchase History | 1 |

**I duplicati sono reali.** La scheda Tools del singolo figlio ne mostra una sola voce e
inganna; la vista completa è in **Copilot for finance and operations apps → Tools**, che
elenca *tutte* le registrazioni con destinatario e stato.

Sono **11 registrazioni** (9 Flow + 2 MCP):

| strumento | available to | Enabled |
|---|---|---|
| MCP Dynamics 365 ERP RDA | RDA Creation Assistant | On |
| MCP Dynamics 365 ERP Procurement & MMD | RINA Procurement & MMD | On |
| RINA - Check Vendor | RDA Creation Assistant | On |
| RINA - Create RDA Draft | RDA Creation Assistant | **Off** |
| RINA - Category Ledger Accounts | RINA Procurement & MMD | On |
| RINA - Get Item Accounting Types | RDA Creation Assistant | On |
| RINA - Get Item Accounting Types | RINA Procurement & MMD | On |
| **RINA - Search Items** | **Copilot for f&o apps (il padre)** | **Off** |
| **RINA - Search Items** | RDA Creation Assistant | On |
| **RINA - Search Items** | **Copilot for f&o apps (il padre)** | **On** |
| **RINA - Search Items** | **RINA Procurement & MMD** | **Off** |

I conti con la solution tornano esattamente: 4 registrazioni di Search Items + il flusso
= i 5 componenti contati; 2 registrazioni di Get Item Accounting Types + il flusso = 3.
Il conteggio della solution era un indicatore corretto.

### Quello che questa tabella dice davvero

**Search Items è registrato quattro volte, su tre destinatari diversi, e lo stato è
incoerente:**

- la registrazione verso **RINA Procurement & MMD Assistant** — l'agente che fa le
  ricerche, e le cui istruzioni contengono tutte le regole sulle parole chiave italiane
  — è **spenta**;
- due registrazioni sono intestate direttamente al **padre**, una spenta e una accesa;
- quella accesa sul padre è la più recente (22 giorni fa).

Se la ricerca oggi viene invocata dal padre e non dal figlio, allora **le parole chiave
le compone il padre**, che non ha il blocco di istruzioni scritto apposta per costruirle.
È una spiegazione concreta e verificabile del motivo per cui **le keyword cambiano a ogni
esecuzione** — il difetto aperto da settimane.

Da verificare prima di toccare qualunque cosa, perché è una modifica delicata su un
agente managed e sul comportamento in esercizio.

### b-bis) Tre flussi su sei non sono agganciati a questo agente

Confronto fra i flussi nella solution e gli strumenti registrati sul figlio:

| flusso nella solution | è uno strumento del figlio? |
|---|---|
| RINA - Search Items | sì |
| RINA - Get Item Accounting Types | sì |
| RINA - Get Category Accounts | probabilmente, sotto il nome *RINA - Category Ledger Accounts* (da confermare) |
| RINA - Check Vendor | **no** |
| RINA - Create RDA Draft | **no** |
| RINA - Item Purchase History | **no** |

Spiegazione plausibile: appartengono a **RDA Creation Assistant**, il secondo agente
figlio, che risulta **Enabled = Off**. Da verificare aprendo i suoi strumenti.

Conta per il rilascio: se quei flussi servono a un agente spento, vanno in produzione
inattivi — e va deciso se è voluto.

### c) Le due environment variable segnalate sono estranee

*System Optimization Dashboard API config* e *QEA Fabric MCP Server URL*, entrambe
vuote, nessun rapporto con il Procurement Assistant. Non toccate.

---

## Conseguenza sulla checklist di rilascio

Alla lista delle cose da fare a mano in produzione — istruzioni del padre,
collegamento padre-figlio, modello, Express mode, condivisioni — si aggiungono
**con certezza**:

1. ricaricare le knowledge e verificarle *Ready*;
2. creare o ripuntare le environment variable eventualmente usate dai flussi;
3. ripuntare le 4 connection reference e verificare i 6 flussi accesi.
