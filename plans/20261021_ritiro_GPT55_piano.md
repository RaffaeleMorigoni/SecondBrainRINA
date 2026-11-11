# Ritiro di GPT-5.5 Chat — come ci prepariamo

**Il fatto:** nella pagina dell'agente c'e' l'avviso
*"GPT-5.5 Chat will retire on 10/30/2026"*. Oggi e' il 21/10: **restano nove giorni**.
Il modello attuale dell'orchestratore e' esattamente quello.

---

## 1. Perche' per noi non e' un cambio banale

Tutto il lavoro delle ultime due settimane e' **tarato su quel modello**: la lingua della
risposta, il prefisso `MMD | `, l'elenco uno per riga, la regola della risposta unica, il divieto
di chiedere la lingua, il formato senza parentesi. Nessuna di queste cose e' garantita dal codice:
sono comportamenti che **quel** modello tiene perche' glielo abbiamo chiesto in quel modo.

Cambiare modello puo' far saltare proprio quelle. Non e' un aggiornamento tecnico: e' un
cambio di motore sotto istruzioni scritte a mano.

---

## 2. Le opzioni che offre il selettore

**OpenAI**

| modello | nota |
|---|---|
| **GPT-5 Chat** | *Great for most tasks*. Stessa famiglia "chat", non in preview |
| GPT-5 Auto | **Preview** — alterna chat e reasoning |
| GPT-5 Reasoning | **Preview** — massima profondita' |
| GPT-4.1 | generazione precedente |
| GPT-5.5 Chat | **attuale, in ritiro il 30/10** |
| **GPT Chat Latest** | segue sempre l'ultimo disponibile |

**Anthropic**: Claude Sonnet 4.6, Claude Opus 4.6 / 4.7 / 4.8.

### Come la vedo

- **`GPT-5 Chat` e' il candidato naturale**: stessa famiglia e stesso profilo d'uso di quello che
  abbiamo, non e' in preview, non sparisce il 30.
- **`GPT Chat Latest` e' comodo ma rischioso per noi**: ci toglie il problema dei ritiri futuri,
  ma fa cambiare il motore **senza preavviso**. Su un agente il cui comportamento e' gia'
  instabile fra una pubblicazione e l'altra, significa non sapere piu' se una regressione e'
  nostra o del modello. **Lo sconsiglio finche' l'agente non e' stabile.**
- **I due Preview non vanno messi in un ambiente che sta per andare in esercizio.**
- **Claude** e' un cambio di famiglia: andrebbe valutato, ma non in nove giorni e non insieme al
  resto.

---

## 3. Il piano, in ordine

### Passo 1 — censimento: FATTO il 21/10

Nell'ambiente ci sono **52 agenti**, ma i nostri sono **due**. Gli altri cinquanta sono di SYSTEM
(D365 Sales Agent, Expense Agent, Customer Service…), installati con Dynamics.

| agente | cosa e' | modello |
|---|---|---|
| `Copilot for finance and operations apps` | orchestratore (il nostro padre) | **GPT-5.5 Chat — in ritiro** |
| `RINA Procurement & MMD Assistant` (`dfd051a3…`) | bot standalone | **GPT-5.5 Chat — in ritiro** |
| `RINA Procurement & MMD Assistant` (`b4a1f29d…`) | agente connesso, nodo del padre | **eredita dal padre** |
| `RDA Creation Assistant` | agente connesso, nodo del padre (oggi **Off**) | **eredita dal padre** |

**Gli agenti connessi non hanno un proprio selettore di modello**: in *Advanced* ci sono solo
Condition e Priority. Quindi seguono il padre, e cambiando il padre cambiano anche loro.

**Da cambiare a mano ci sono quindi solo due agenti.** Meno lavoro del previsto.

**Un dato che orienta la scelta:** ho controllato a campione un agente di sistema
(`Expense Entry Agent`): usa **GPT-5 Chat**, e non ha nessun avviso di ritiro. Cioe' il default
di Microsoft per gli agenti in questo stesso ambiente e' gia' GPT-5 Chat. I nostri due sono su
GPT-5.5 per una scelta fatta a suo tempo, non perche' fosse il default.

### Passo 2 — fissare la suite prima di toccare (un'ora)
**Prima** del cambio, rilanciare la suite e salvare le risposte come riferimento. Serve il
"prima" scritto, altrimenti dopo non sappiamo cosa e' cambiato:

1. ricerca in italiano (manutenzione attrezzature, RSSE) — formato, elenco, lingua
2. ricerca in francese su RSSE — lingua, nessuna parentesi, disponibilita' in societa'
3. **due turni consecutivi in francese** — il follow-up e' il caso fragile
4. CAPEX/OPEX concettuale e ancorata a un acquisto — knowledge e rimando a Finance
5. contabilizzazione servizi — risposta unica, turno che si chiude
6. una ricerca su RSPA — ramo "da richiedere" e traduzioni

### Passo 3 — cambiare in un momento controllato
**Non il 29.** Un giorno in cui ci siamo e possiamo guardare, con il SAL che sa che stiamo
cambiando. Prima l'orchestratore, suite, poi gli altri agenti.

### Passo 4 — rimisurare anche i tempi
Il modello incide sulla latenza, e noi siamo a **1m01s su un limite che scatta verso 1m40s**
(vedi `20261021_stato_UAT.md`). Un modello piu' lento ci riporta dentro il timeout.
**Va rimisurata la durata, non solo la correttezza.**

### Passo 5 — tenere il "come era"
Backup delle istruzioni di tutti gli agenti **prima** del cambio, come fatto il 21/10
(`plans/backup/`). Se il nuovo modello richiede di riscrivere qualche regola, serve sapere da
dove si parte.

---

## 4. Cosa chiederei a RINA

1. **Quando** possiamo fare il cambio: serve una finestra concordata, non un venerdi' sera.
2. Se hanno **indicazioni di gruppo** sul modello da usare (puo' essere una scelta IT, non nostra).
3. Che sappiano che **dopo il cambio la suite va rifatta dal business**, non solo da noi: alcune
   cose che controlliamo (tono, lingua, formato) le vedono solo loro usandolo davvero.

---

## 5. La cosa da non fare

Cambiare modello **adesso**, a fine giornata, dopo aver pubblicato sei modifiche. Se qualcosa si
rompe domani non sapremmo se e' il modello o una delle modifiche di oggi.
