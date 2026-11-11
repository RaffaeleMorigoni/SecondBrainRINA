# Chi ha usato l'agente negli ultimi giorni — e cosa ha trovato

Richiesta di Giorgio per il SAL: verificare nel monitoring se qualche utente RINA ha fatto test
negli ultimi 3 giorni.

**Fonte giusta:** `Monitor → Analytics → Sessions`. **Non** `Activity`: quella vista si ferma al
22/10 e mi aveva fatto concludere, sbagliando, che non ci fosse nessuna attivita'.

---

## La risposta: si', hanno provato. E non ha funzionato.

6 sessioni negli ultimi 7 giorni (22-28/10), **3 delle quali oggi**.

| quando | durata | messaggi | esito dichiarato | cosa e' successo davvero |
|---|---|---|---|---|
| **28/10 13:58** | 00:12 | 5 | Unengaged | *"You don't have access to talk to this bot"* ×5 |
| **28/10 08:13** | 00:15 | 5 | Unengaged | *"You don't have access to talk to this bot"* ×5 |
| **28/10 08:06** | 06:29 | 9 | **Resolved** | richiesta di **connessione credenziali**, ripetuta 3 volte |
| 24/10 11:50 | 00:02 | 0 | Unengaged | — |
| **24/10 11:41** | 11:46 | 11 | Abandoned (User error) | **due `FlowActionTimedOut`**, poi abbandono |
| 22/10 13:09 | 02:30 | 4 | Resolved | — |

---

## I tre modi diversi in cui si sono bloccati

### 1. Timeout — 24/10, utente italiano

> **11:41** *devo acquistare una consulenza informatica*
> **11:43** Sorry, the bot can't talk for a while… **Error code: FlowActionTimedOut**
> **11:44** *devo acquistare un monitor*
> **11:45** **FlowActionTimedOut**
> **11:49** *attiva agente mms*

Due tentativi, due timeout, poi l'utente ha provato a sbloccarlo da solo e ha lasciato perdere.
**Il timeout e' tornato**, nonostante Express mode e la parallelizzazione del 21/10.
Prova: `prove_uat/20261024_flowactiontimedout_utente.jpg`

### 2. Richiesta di credenziali — 28/10 mattina

> **08:07** *mmd what is the product ?*
> **08:07** *Let's get you connected first… **Open connection manager to verify your credentials**.*
> **08:08** *what code is this ?*
> **08:08** stessa identica card

L'utente insiste, l'agente ripete tre volte la stessa richiesta di connessione.
**E la sessione risulta "Resolved"**: il dato aggregato dice che e' andata bene, la trascrizione
dice il contrario. Da tenere a mente ogni volta che guardiamo le statistiche.
Prova: `prove_uat/20261028_connection_manager.jpg`

### 3. Nessun accesso — 28/10, due sessioni

> **You don't have access to talk to this bot, contact the owner.** (×5, in entrambe)

Qualcuno non e' proprio autorizzato a parlare con l'agente.
Prova: `prove_uat/20261028_no_access.jpg`

---

## Cosa ne ricavo

**Il problema oggi non sono le risposte dell'agente: e' che la gente non riesce ad arrivarci.**
Tre sessioni su sei degli ultimi giorni si sono fermate prima di qualunque ricerca, per
autorizzazioni e connessioni. Una quarta e' morta sul timeout.

Tutto il lavoro di messa a punto delle risposte (lingua, formato, CAPEX/OPEX) non e' ancora
arrivato in mano a nessuno.

**Tre cose da portare al SAL, in quest'ordine:**

1. **Permessi**: chi sono gli utenti che ricevono *"You don't have access"* e perche'. E' la
   barriera piu' banale e la piu' bloccante.
2. **La connessione** richiesta dal *connection manager*: va capito quale strumento la pretende e
   se si puo' pre-autorizzare, altrimenti ogni utente nuovo sbatte li'.
3. **Il timeout e' tornato** il 24/10 su domande normali ("una consulenza informatica",
   "un monitor"). Va riaperto come difetto.

E un avviso che non c'entra con la domanda di Giorgio ma scade prima: **GPT-5.5 Chat va in ritiro
il 30/10**, dopodomani, e il modello non e' ancora stato cambiato su nessuno dei due agenti.
