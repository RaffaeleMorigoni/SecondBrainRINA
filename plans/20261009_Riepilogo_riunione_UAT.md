# UAT dell'8 ottobre — punto per la riunione

Aggiornato al 09/10/2026, ore 17 (seconda revisione). Fonti: registro UAT (copia HUDi), user story DevOps 1812 e i suoi
14 task, suite di regressione da 50 casi eseguita tre volte.

---

## In una riga

Otto segnalazioni chiuse e verificate, tre avanzate a meta', il resto fermo per ragioni che
sappiamo dire. I due difetti piu' fastidiosi dei tester sono risolti, e in entrambi i casi la causa
non era dove la cercavamo: i codici vecchi li passava il flusso di ricerca, il doppio messaggio lo
scriveva l'orchestratore di progetto. Nel pomeriggio ho riletto tutti i task sulla loro descrizione
invece che sulla segnalazione del tester, e su tre di essi il giudizio del mattino era sbagliato.

---

## 1. Chiuso e verificato

| Cosa | Prova |
|---|---|
| **Codici PTEC/legacy non piu' esposti** | screenshot su task 2335: ISO 9001 risponde `PS101002.0002`, `Npcons410` sparito |
| **Stati di rilascio in positivo** | screenshot su task 2331: «Articoli disponibili nella tua societa» / «presenti in Dynamics ma da richiedere» |
| **Niente tetto ai cinque risultati** | ricerche ampie elencano tutti i codici pertinenti |
| **Prefisso «MMD \|»** | 46 risposte su 50 nella suite |
| **Timeout del flusso di ricerca** | da 96 a 23 secondi, errore `FlowActionInternalServerError` sparito |
| **Contabilizzazione senza categoria assegnata** | `MA101008.0002` risponde: tipo 1, conto 410000, Goods. Flusso da 41 a 2 secondi |
| **Doppio messaggio eliminato** | task 2333: 7 turni consecutivi in F&O, una sola risposta ciascuno |
| **Guida alla verifica delle deleghe** | task 2327: elenca i 3 criteri e le pagine di F&O dove controllarli, senza presumere autorizzazioni |

**La storia da raccontare.** Per due volte abbiamo provato a vietare i codici vecchi scrivendolo
nelle istruzioni dell'agente, senza risultato. La causa era altrove: il flusso di ricerca
costruiva ogni voce come `Npcons410 - Consulenza per certificazioni`, cioe' gli passava il codice
gia' scritto. Una riga cambiata nel flusso ha chiuso il caso al primo colpo, e la ricerca per
categoria continua a funzionare.

---

## 2. A meta'

| Cosa | Fatto | Manca |
|---|---|---|
| Presentazione dei risultati | elenco completo | resiste la frase «Vuoi l'elenco completo?» |
| Formattazione | struttura a due sezioni recuperata | grassetto e stacco non garantiti |
| Mapping PTEC | i codici vecchi non si vedono piu' | l'associazione segnalata da Emanuela e le corrispondenze multiple |

---

## 3. Fermi, e perche'

- **Marcatore «1»** — 9 risposte su 50. Non lo scrive il modello, lo inserisce il motore delle
  citazioni sulle knowledge. Nelle impostazioni dell'agente non c'e' un interruttore.
- **Codici non piu' validi** — serve sapere da RINA come un codice viene marcato come obsoleto in
  F&O. Se il dato esiste, il filtro e' semplice.
- **Procedura di richiesta articolo** — la distinzione fra disponibile e da richiedere ora c'e'.
  Quello che manca e' dire *come* si richiede: serve la procedura ufficiale da RINA.
- **Domanda di conferma sulle alternative** — l'agente spiega le differenze fra i codici ma sceglie
  da solo invece di chiedere quale usare. Da bilanciare con la richiesta opposta di fare meno
  domande.
- **Lingua** — oggi l'agente segue la lingua dell'utente. Il task chiede italiano fisso salvo
  richiesta esplicita. E' una scelta da fare, non un difetto.

---

## 4. In carico a RINA

Linea guida AFC sulla capitalizzazione, documento educativo per gli utenti, profilazione di lettura
dei materiali, completamento delle anagrafiche in UAT, governo degli aggiornamenti delle descrizioni
estese.

---

## 5. Stato della board

Su DevOps, user story 1812: **tutti i task commentati**, ciascuno con la prova in allegato.
Portati a Done: **2313, 2327, 2331, 2332, 2333, 2335**. Restano aperti con diagnosi 2324, 2328,
2329, 2330, 2334; 2325 e 2326 aspettano una decisione RINA.

I commenti del mattino su **2327, 2330 e 2333** sono stati rettificati: li avevo scritti leggendo
la riga del registro invece della descrizione del task, e in tutti e tre i casi il requisito reale
era gia' soddisfatto o lo e' diventato nel frattempo.

Un'avvertenza pratica: Chiara stava lavorando sugli stessi task in parallelo. Niente e' stato
sovrascritto — solo commenti aggiunti — ma conviene allinearsi su chi muove gli stati.

---

## 6. Le due cose da decidere

**Completezza o struttura.** I tester chiedono di non mettere soglie ai risultati. Ma e'
plausibilmente proprio l'elenco lungo a far riassumere il modello, che nel riassunto perde la
seconda sezione. Non si possono avere entrambe senza un intervento dedicato.

**Formato.** Lo confermiamo come variabilita' nota della piattaforma, oppure si stima un intervento
che tolga al modello la decisione sull'impaginazione. Abbiamo provato tutte le leve disponibili —
istruzioni, testo gia' formattato dal flusso, campo dedicato — e nessuna lo rende deterministico.

---

## Nota sui numeri della suite

Tre esecuzioni: Answer quality 82 / 78 / 80 per cento, Meaning match 52 / 60 / 52. **Oscillano senza
seguire le modifiche: non usarli come indicatore di avanzamento.** L'unico che si e' mosso in modo
pulito e' la presenza della seconda sezione, da 1 a 7 a 11 risposte su 50.
