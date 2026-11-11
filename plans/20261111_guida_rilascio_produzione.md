# Rilascio in produzione — Procurement & MMD Assistant

**Data:** 11/11/2026
**Stato:** guida preparatoria, nessuna operazione eseguita

---

## 0. Il punto che cambia tutto il piano

Questo agente **non è un singolo oggetto esportabile**. È composto da cinque strati, e solo due viaggiano dentro una solution:

| strato | si esporta? | come arriva in prod |
|---|---|---|
| Agente figlio custom (`b4a1f29d-c9f8-4c8d-889c-f1906ea65b5f`) | **sì** | solution managed |
| Flussi Power Automate (`RINA - Search Items` + gli altri) | **sì** | solution managed |
| **Istruzioni dell'orchestratore padre** (`Copilot for finance and operations apps`) | **no** | **da ricopiare a mano** |
| **Knowledge** (Excel codici, guida CAPEX/OPEX) | **no** | **da ricaricare a mano** |
| Condivisioni agli utenti, modello, Express mode, connection reference | **no** | **da riconfigurare per ambiente** |

L'orchestratore padre è l'agente **di sistema** di F&O: esiste già in ogni ambiente, con le sue istruzioni vuote o di default. Le nostre 7.989 caratteri di istruzioni sono configurazione di *quell'ambiente*, non un artefatto nostro. In produzione vanno riscritte.

**Conseguenza pratica: più della metà del lavoro di rilascio è manuale, e quindi sbagliabile.** Il piano sotto serve a renderlo ripetibile.

---

## 1. Blocchi da chiudere PRIMA di fissare la data

### 1.1 Il limite 8.000 caratteri — bloccante

Le istruzioni del padre sono a **7.989 / 8.000**. Undici caratteri di margine.

Perché è bloccante e non un dettaglio: dopo il go-live il primo difetto segnalato dal business si corregge in quelle istruzioni, e **non c'è spazio per correggerlo**. Peggio: il campo tronca a 8.000 **senza avvisare**, quindi un incollaggio un po' lungo decapita silenziosamente l'ultimo blocco — e l'ultimo blocco è dove stanno le regole che contano.

Azione: prima del rilascio, una passata di compattazione delle istruzioni del padre per riportarle sotto **7.000**, a parità di comportamento, verificata con la suite di regressione. Serve margine di manovra, non estetica.

### 1.2 Risposte non deterministiche — da dichiarare, non bloccante

Le parole chiave con cui l'agente interroga l'anagrafica cambiano a ogni esecuzione: la stessa domanda può dare esiti diversi. Non impedisce il rilascio (l'agente è uno strumento di supporto alla scelta del codice, non un calcolo contabile), ma va **dichiarato formalmente** nel verbale di accettazione, altrimenti al primo caso discordante diventa un difetto post-go-live.

### 1.3 Il modello "Latest" si muove sotto i piedi — rischio da presidiare

L'agente gira su **GPT Chat Latest**. Non è una versione fissata: è un'etichetta che Microsoft fa avanzare. Il 30 ottobre GPT-5.5 Chat è stato ritirato con preavviso breve e abbiamo dovuto migrare; **succederà di nuovo**, e in produzione significa che il comportamento dell'agente può cambiare in un giorno senza che nessuno abbia toccato niente.

Non è evitabile (non ci sono modelli pinnati a disposizione). Va messo a contratto come rischio noto, con un presidio: rilanciare la suite di regressione dopo ogni avviso di ritiro modello.

### 1.4 Selettività dopo il fix onsite — aspettare il riscontro

Il fix del 10/11 ha reso l'agente più selettivo: la domanda sulla manutenzione è passata da sei codici proposti a uno. Per una richiesta precisa è giusto, per una generica potrebbe essere troppo stretto. **Ho chiesto riscontro ai tester e non è ancora arrivato.** Non andrei in produzione senza quella conferma: è l'unica modifica recente di cui non conosco l'impatto sul campo.

### 1.5 Limiti noti da mettere per iscritto, non da risolvere

- Domanda in una lingua con interfaccia F&O in inglese: può rispondere in inglese.
- Ricerca per parole in francese: intermittente.
- Società con solo codici legacy (RCSM, RSPA): l'agente non trova codici MMD nuovi perché **non ci sono**. Non è un difetto suo.

---

## 2. Prerequisiti sui DATI di produzione

Questi sono i veri fallimenti silenziosi: tutto si installa, niente funziona, e nessuno capisce perché. **Da verificare in prod prima del go-live, non dopo.**

1. **Codici MMD nuovi presenti** nelle legal entity dove lavorano gli utenti. Se una società ha solo codici legacy, l'agente lì è inutile — e va detto prima.
2. **`ProductSearchName` popolato in italiano.** La ricerca gira su quel campo, per prima parola. Se in prod è vuoto o in altra lingua, la ricerca non trova nulla.
3. **`ProductTranslations`** presenti: sono la via d'uscita sul problema lingua.
4. **Descrizioni delle categorie nuove.** In UAT le 591 categorie nuove esistono ma sono mute: se in prod è lo stesso, la ricerca aggancia solo le legacy.
5. **Lavoratore collegato all'utenza** per chi userà il flusso RDA: senza quello la RDA non parte e l'Originator resta vuoto.
6. **Rilasci inter-societari** (parametro Cross Company) coerenti con UAT, o le etichette *disponibile / da richiedere* diranno il falso.

---

## 3. Procedura, nell'ordine

### Fase A — preparazione (giorni prima)

1. **Congelare UAT.** Dal momento in cui si fa l'export, nessuna modifica in UAT: ogni ritocco successivo è una divergenza da riportare a mano.
2. **Baseline documentata.** Snapshot di: istruzioni del padre (testo integrale), istruzioni del figlio, descrizioni di tutti gli strumenti e del trigger, elenco knowledge, modello selezionato, stato Express mode. Questo è l'unico modo per ricostruire la configurazione manuale senza dimenticare pezzi.
3. **Suite di regressione eseguita in UAT** e esiti registrati: è il metro di confronto, altrimenti in prod non sai distinguere un difetto nuovo da uno preesistente.
4. **Solution.** Verificare che agente figlio e flussi siano in una solution *unmanaged* dedicata (non in Default). Se sono in Default — probabile — vanno aggiunti a una solution nuova prima di poter esportare.
5. **Connection reference.** Censire le connessioni usate dai flussi e decidere **con quale utenza di servizio** gireranno in produzione. Questa è una decisione da prendere con RINA, non da noi: in UAT girano sull'utenza `C_110912`, in prod serve un'utenza di servizio dedicata con i permessi F&O corretti e una password che non scade.

### Fase B — rilascio

6. Export della solution da UAT **come managed**.
7. Import in produzione. Durante l'import si mappano le connection reference sulle connessioni di prod (create prima, al punto 5).
8. **Riscrittura delle istruzioni dell'orchestratore padre** in prod, dalla baseline. Attenzione: incollare, poi **rileggere l'ultimo blocco** per verificare che non sia stato troncato.
9. **Ricaricamento della knowledge.** Verificare che risultino *Ready* (non basta caricarle: devono completare l'indicizzazione).
10. **Selezione del modello** e **Express mode** da riattivare: sono per ambiente e non seguono la solution.
11. **Verifica delle variabili globali** lingua: devono esistere e comportarsi come in UAT.
12. **Publish.**

### Fase C — abilitazione utenti

13. **Due autorizzazioni distinte, entrambe necessarie:** la condivisione dell'agente in Copilot Studio **e** l'accesso a F&O. Senza la prima l'utente vede *"You don't have access to talk to this bot"* anche se in F&O entra regolarmente.
14. Partire con un **gruppo pilota ristretto** (5-10 persone, le stesse che hanno testato), non con tutti. Allargare dopo una settimana pulita.

### Fase D — verifica in produzione

15. Rilanciare **la stessa suite di regressione** usata in UAT, in prod, con un'utenza reale e la società giusta. Confronto riga per riga con la baseline UAT.
16. Controllare in Power Automate che i flussi partano davvero e con quali parametri: in prod i permessi e le connessioni sono diversi, ed è lì che si rompono.

---

## 4. Rollback

Da decidere **prima**, non durante.

- **Solution managed:** si disinstalla, e porta via agente figlio e flussi. Pulito.
- **Istruzioni del padre:** non c'è versioning. Il rollback è reincollare la baseline. **Per questo il punto A.2 non è burocrazia:** senza lo snapshot del testo precedente non si torna indietro.
- **Via di fuga immediata:** togliere la condivisione dell'agente agli utenti. Spegne l'accesso in un minuto senza smontare niente, ed è la prima mossa se in prod qualcosa va storto.
- **Knowledge:** rimuovibili singolarmente. Ricordare che competono con la ricerca: una knowledge di troppo fa rispondere l'agente senza interrogare F&O.

---

## 5. Dopo il go-live

- **Il monitoraggio conserva 29 giorni.** Oltre quella finestra una segnalazione non è più indagabile: va detto ai tester, perché cambia il valore di segnalare subito.
- Guardare **Analytics > Sessions**, non Activity. E attenzione: l'esito *Resolved* può nascondere una sessione fallita.
- Dall'Analytics **non si ricava chi ha testato**: si ottengono volumi e DAU, non identità. Se serve sapere chi, va chiesto.
- Presidiare gli avvisi di ritiro modello (vedi 1.3).

---

## 6. Da chiarire con RINA prima di procedere

1. **Chi esegue il rilascio** — noi, il loro team Power Platform, o a quattro mani? Cambia tutta la logistica.
2. **Esiste già un ambiente di produzione F&O + Power Platform** configurato, o va predisposto?
3. **Utenza di servizio** per le connessioni dei flussi in produzione: chi la crea, con quali permessi.
4. **Chi firma l'accettazione** e su quale perimetro: in particolare se i limiti noti del punto 1.5 e il non determinismo del 1.2 sono accettati.
5. **Data obiettivo** e finestra: il rilascio ha una parte manuale, meglio non farla di venerdì sera.

---

## 7. Il mio giudizio in una riga

Tecnicamente si può rilasciare. **Non lo farei prima di aver fatto tre cose:** liberato spazio nelle istruzioni del padre (1.1), ricevuto il riscontro dei tester sulla selettività (1.4), e verificato i prerequisiti sui dati di produzione (sezione 2). Sono giorni, non settimane. Senza la prima, il primo difetto post-go-live non è correggibile.
