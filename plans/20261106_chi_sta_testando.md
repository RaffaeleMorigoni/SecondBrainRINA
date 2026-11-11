# Chi sta testando l'agente — dato al 06/11/2026

Fonte: **Copilot Studio > Monitor > Sessions** dell'orchestratore, scaricate tutte le giornate
disponibili (16 file CSV). La retention e' di **29 giorni**: il dato copre **9/10 - 6/11**,
prima non e' piu' recuperabile.

## Il limite da sapere subito

**Analytics non espone chi ha fatto la sessione.** Ne' la tabella a video ne' il CSV hanno un
campo utente: le colonne sono SessionId, data, esito, turni, trascrizione, primo messaggio,
topic, canale, CSAT. **Non si puo' quindi spuntare l'elenco dei 42 nomi.**

Si puo' pero' dire **quanto**, **quando**, **in che lingua** e **su cosa**.

## Quanto si e' testato

**205 sessioni con un messaggio reale**, di cui **46 del 30/10 sono nostre** (la giornata di
prove multilingua). Restano **~159 sessioni** di altri.

| giorno | sessioni | |
|---|---|---|
| 09/10 | 17 | |
| 12-14/10 | 9 | |
| **15/10** | **33** | picco |
| **16/10** | **39** | picco |
| 19/10 | 16 | |
| 21/10 | 15 | |
| 22, 24, 28/10 | 5 | |
| 29/10 | 17 | |
| 30/10 | 46 | **36 nostre** |
| **03/11** | **4** | |
| **05/11** | **2** | |

## Il dato che conta: i test si sono fermati

Dopo il picco di meta' ottobre (15-16/10: **72 sessioni in due giorni**) l'attivita' cala, e
**dal 31/10 e' quasi ferma: 6 sessioni in sette giorni**.

Le ultime, per intero:

```
03/11  Item code for Extra Km on a project?
03/11  devo acquistare un servizio di catering per una evento in direzione
03/11  Which item code should I use for generic project expenses re-billed to the client
05/11  quale articolo devo utilizzare al posto del vecchio Npcons611 ?
05/11  devo aprire una richiesta di acquisto per acquistare delle licenze software per...
```

Sono domande vere, di utenti veri — ma sono **sei in una settimana**.

## In quante lingue (escluso il 30/10, che e' nostro)

| lingua | sessioni |
|---|---|
| inglese | 42 |
| italiano | 39 |
| francese | 29 |
| portoghese | 18 |
| spagnolo | 9 |
| cinese | 2 |
| non classificate | 20 |

**I tester esteri ci sono e sono attivi**: quasi 60 sessioni fra francese, portoghese, spagnolo
e cinese. Non e' un gruppo di soli italiani.

## Esiti

**184 Resolved, 21 Abandoned.** Ma "Resolved" e' un'etichetta automatica e puo' coprire una
sessione fallita ([[rina-monitoring-sessioni-dove-guardare]]): non va letta come tasso di
successo.

## Come avere davvero i nomi

L'unica fonte che identifica l'utente e' **Power Automate**: il trigger di `RINA - Search Items`
riceve `UserEmail` (l'UPN di chi sta chattando). Si puo':

1. aprire le run del flusso e leggere `UserEmail` run per run — preciso ma lento;
2. oppure, meglio, **chiedere a RINA il log degli accessi** all'agente, che hanno loro.

In alternativa, se la domanda e' *"chi non ha mai provato"*: il modo rapido e' **chiederlo a
loro**, perche' sei sessioni in una settimana su 42 persone convocate e' un dato che parla da
solo e non ha bisogno dei nomi.

## Da portare in riunione

- **Il test e' sostanzialmente fermo dal 31/10**: sei sessioni in sette giorni.
- Il grosso si e' concentrato il **15-16 ottobre** e poi si e' spento.
- **Chi prova non e' solo italiano**: francese, portoghese, spagnolo e cinese sono rappresentati.
- **I nomi non sono ricavabili** dagli strumenti che abbiamo: vanno chiesti a RINA.

CSV in `Downloads/Sessions *.csv`.

---

# Tentativo di leggere `UserEmail` run per run: **strada chiusa**

Provato come concordato. Non funziona, per tre motivi indipendenti:

1. **Gli input del trigger non sono recuperabili.** Nella run il pannello dice *"Error loading
   inputs"* / *"Error loading outputs"*, e chiedendo il contenuto direttamente all'API risponde
   `InternalServerError`. Vale per tutte le run provate, non e' un caso isolato.
2. **Nessuna azione del flusso usa `UserEmail`.** Le 35 azioni sono tutte su prodotti,
   traduzioni e categorie (`Filtra parole`, `Lista prodotti 1-4`, `Componi clausole`,
   `Clausole traduzioni LE`, `Lista categorie`…). Il flusso riceve l'email ma non la interroga
   mai, quindi non compare in nessun input visibile.
3. **Analytics da' solo numeri aggregati**, mai nomi: ne' la tabella, ne' il CSV, ne' il pannello
   *Active users*.

## Ma Activity ha dato numeri migliori di Sessions

**Monitor > Activity, ultimi 7 giorni (31/10 - 6/11):**

| | |
|---|---|
| Conversation sessions | **9** — **-88%** |
| Engagement | 67% — **-25%** |
| **Average DAU** (utenti attivi al giorno) | **1** — **-91%** |
| Total reactions | **0** |

E il riepilogo automatico di Microsoft, testuale:

> *"Agent sessions and engagement sharply declined with no autonomous runs or user feedback
> recorded. Total sessions dropped 88% to 9."*

**Dal grafico degli utenti attivi giornalieri** si leggono i soli due giorni con attivita':

- **3 novembre: 6 utenti distinti**
- **5 novembre: 3 utenti distinti**
- tutti gli altri giorni: **zero**

## Una cautela sull'interpretazione

Il grafico mensile mostra per ottobre **circa 230 utenti attivi**: non sono i nostri tester.
L'orchestratore `Copilot for finance and operations apps` e' il Copilot **generale** di F&O, e
conta anche chi lo usa per tutt'altro. Il dato affidabile per noi e' quello **giornaliero
recente**, dove i numeri sono piccoli e i messaggi (gia' letti) sono tutti su MMD e RDA.

## Il punto, in una riga

**Non si possono spuntare i 42 nomi**, ma non serve: negli ultimi sette giorni l'agente e' stato
usato in **due soli giorni**, da **6 e 3 persone**, con **zero reazioni** lasciate. I nomi, se
servono davvero, vanno chiesti a RINA.
