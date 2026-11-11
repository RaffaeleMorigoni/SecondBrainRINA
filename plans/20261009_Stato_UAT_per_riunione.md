# Stato del registro UAT dell'8 ottobre — punto per la riunione

Aggiornato al 09/10/2026, ore 17 (seconda revisione, dopo le prove sui task DevOps). Fonte: `20261008_UAT_RINA_FO_INTERNO_HUDi.xlsx` (copia HUDi;
l'originale dei tester non e' stato toccato). 34 righe, piu' 3 aggiunte da noi.

Come leggerlo: **Chiuso** = corretto e verificato in F&O o sulla suite. **Fatto, da verificare** =
l'intervento c'e' ma non ho ancora la prova sul campo. **Aperto** = non risolto, con la diagnosi
dove ce l'ho. **RINA** = non dipende da noi.

---

## 1. Chiusi e verificati — 9 righe

| # | Segnalazione | Cosa e' stato fatto | Prova |
|---|---|---|---|
| 3 | Massimo cinque suggerimenti | Tolto il tetto, regola "elenca tutti i risultati pertinenti" | Suite + F&O: ricerche ampie elencano tutto |
| 7 | Il prefisso «MMD \|» deve esserci sempre | Spostato nel campo Response formatting | 46 risposte su 50 nella suite |
| 13 | `FlowActionInternalServerError` su CAPEX | Riscritto il filtro di `Lista nature`, filtro Hybrid spostato a valle | Flusso da 96 a 23 secondi, risposta corretta |
| 17 | «Nessun articolo non rilasciato» ambiguo | Stato di rilascio scritto in positivo | F&O: «Disponibili in Dynamics e richiedibili» |
| 19 | Struttura a due sezioni approvata | Regola rimessa nelle istruzioni dopo che un mio taglio l'aveva persa | Da 1 a 11 risposte su 50, e sul caso canonico |
| 31 | Codici PTEC/legacy esposti | **Corretto nel flusso**: `Seleziona categorie` restituiva `codice - descrizione`, ora solo la descrizione | F&O: `Npcons410` sparito, ricerca per categoria intatta |
| 28 | Doppio messaggio per lo stesso turno | Label e regola di risposta unica messe nel Response formatting **dell'orchestratore di progetto**, non del figlio | F&O: 7 turni consecutivi, una sola risposta ciascuno |
| 10 | Deleghe su altra Legal Entity | Nessun intervento necessario: l'agente gia' guida alla verifica senza presumere autorizzazioni | F&O: elenca i 3 criteri e le pagine di F&O dove controllarli |
| 12, 23, 26 | Casi gia' approvati dai tester | Comportamento mantenuto | Confermati nelle riesecuzioni |

**Il punto da raccontare sulla 28**: due settimane di tentativi sulle istruzioni del figlio non
avevano chiuso il doppio messaggio, perche' il secondo messaggio non lo scriveva lui. Bastava
scrivere la regola nell'agente padre.

**Il punto da raccontare sulla 31**: non era il modello a disobbedire. Era il flusso a passargli i
codici vecchi. Due giri di divieti nelle istruzioni non avevano funzionato; una riga cambiata nel
flusso ha chiuso il caso.

---

## 2. Fatti, da verificare — 4 righe

| # | Segnalazione | Cosa e' stato fatto | Cosa manca |
|---|---|---|---|
| 9 e 21 | Non proporre il cambio di Legal Entity come rimedio | Regola nelle istruzioni | Riprodurre il caso in UAT |
| 11 | Dire cosa distingue le alternative | Regola nelle istruzioni | Riprodurre il caso in UAT |
| 35 | Nominare la societa invece di «la tua societa» | Regola nel campo formato e nelle istruzioni, pubblicata | Nelle prove la risposta era troppo scarna per mostrarla |

---

## 3. Aperti, con diagnosi — 6 righe

| # | Segnalazione | Dove siamo |
|---|---|---|
| 20 | Marcatore «1» isolato | **Non chiuso.** 9-10 risposte su 50. Non lo scrive il modello: lo inserisce il motore delle citazioni sulle knowledge. Nessuna istruzione lo raggiunge, e in Copilot Studio non c'e' un interruttore nelle impostazioni dell'agente. Va cercato sulla configurazione delle singole fonti |
| 33 | Caso canonico senza `SE152001.0001` e senza sezioni | **Sistematico, 4 prove su 4.** La run del flusso dimostra che il dato c'e': `otherlegalentities` contiene l'articolo, `items` ne contiene 35. E' il modello che riassume e butta la seconda sezione. L'ipotesi che dipendesse dal numero di articoli e' stata testata e **smentita** (con 11 articoli va uguale) |
| 2 | «Vuoi l'elenco completo?» | Parziale: l'elenco ora e' completo, ma la frase resiste a un divieto esplicito |
| 29 e 32 | Formato instabile | Dichiarata **variabilita' nota della piattaforma**. Le regole di forma stanno ora in due posti (campo Response formatting + istruzioni): tenerle in uno solo non funziona |
| 34 | L'agente scrive «codice legacy» | Osservato **una volta**. Non corretto di proposito: la regola esiste gia', e inseguire un'osservazione singola e' costato caro oggi |
| 16 | Mostrare anche i richiedibili | Stessa causa della 33 |

---

## 4. Aperti, non ancora affrontati — 5 righe

| # | Segnalazione | Nota |
|---|---|---|
| 14 | Le due etichette generano dubbi sull'utilizzabilita' | Da riformulare con RINA |
| 22 | Riepilogo finale senza conto, categoria e link a F&O | Il dato esiste, va deciso quando mostrarlo |
| 27 | Codici diversi per la stessa domanda | Dipende da societa e stato di rilascio. Marcato APPROFONDIMENTO RINA-HUDi |
| 10, 18 | Delega su altra LE; `MA101021.0001` non piu' valido | Marcati APPROFONDIMENTO RINA-HUDi |
| 30 | Test multilingua | NEXT STEP, prossima sessione UAT |

---

## 5. In carico a RINA — 6 righe

| # | Cosa serve |
|---|---|
| 4 | Documento educativo da collegare alla knowledge |
| 6 | Linea guida AFC ufficiale sulla capitalizzazione CAPEX/OPEX |
| 8 | Definire la profilazione di lettura dei materiali per utente |
| 15 | Completare le anagrafiche in UAT: oggi non tutto e' caricato o rilasciato |
| 24 | Governo degli aggiornamenti delle descrizioni estese |
| 25 | Campo descrizione estesa nell'anagrafica F&O — gia' marcato non fattibile |

---

## Numeri della suite di regressione

Tre esecuzioni da 50 casi fra l'8 e il 9 ottobre:

| | taglio del blocco | ripristino | + regola categorie |
|---|---|---|---|
| Answer quality | 82% | 78% | 80% |
| Meaning match | 52% | 60% | 52% |
| Sezione «da richiedere» | 1/50 | 7/50 | 11/50 |
| Prefisso `MMD \|` | 43/50 | 46/50 | 43/50 |

**Avvertenza da dire in riunione**: i due punteggi aggregati oscillano in una banda senza seguire
le modifiche, e non vanno usati per decidere. L'unico indicatore che si e' mosso in modo pulito e'
la struttura a due sezioni, 1 → 7 → 11.

---

## Le tre decisioni da portare a casa

1. **Tassonomia.** Le categorie in F&O restano denominate con la vecchia sigla. Ora non si vedono
   piu', ma continuano a essere l'unica chiave di ricerca che funziona finche' le categorie nuove
   non avranno le descrizioni. E' un debito di dato, non di agente.
2. **Riga 3 contro riga 33.** I tester chiedono di non applicare soglie ai risultati, ma elencare
   tutto e' plausibilmente cio' che porta il modello a riassumere e perdere la seconda sezione.
   Serve una scelta: completezza o struttura garantita.
3. **Formato.** Confermiamo la variabilita' come nota di piattaforma, oppure si stima un intervento
   dedicato che tolga al modello la decisione sull'impaginazione.

---

## 6. Task DevOps della PBI 1812 — revisione del pomeriggio

Rilettura di tutti i task **sulla descrizione del task**, non sulla riga del registro: su tre di
essi il giudizio del mattino era sbagliato perche' avevo valutato la segnalazione del tester e non
il requisito scritto. Ogni riga qui sotto ha una prova allegata al task in DevOps.

| Task | Requisito | Esito verificato il 09/10 | Stato |
|---|---|---|---|
| 2313 | Label `MMD \|` anche sull'orchestratore | Aggiunta nel Response formatting dell'orchestratore. Presente in 7 turni su 7 | **Done** |
| 2324 | Presentare tutti i risultati gia' recuperati, senza altre domande | Su "consulenza informatica" elenca tutti e 4. Su "manutenzione attrezzature" risponde con **una riga sola** e chiude con "e altri servizi correlati" | In corso |
| 2325 | Mantenere l'italiano, cambiare lingua solo su richiesta esplicita | Italiano 4 volte su 4 con utente italiano; **inglese se l'utente scrive in inglese**. La regola e' volutamente generica | **Decisione RINA** |
| 2326 | Regole AFC per CAPEX/OPEX | La guida in knowledge copre le categorie e viene applicata; manca la policy AFC con le soglie | Chiuso sul perimetro parziale |
| 2327 | Guidare alla verifica delle deleghe senza presumere autorizzazioni | Elenca i 3 criteri e indirizza alle pagine di F&O. **Giudizio del mattino ribaltato** | **Done** |
| 2328 | Esplicitare le differenze **e** chiedere «Quale vuoi utilizzare?» | Differenze esplicitate; la domanda di conferma **non** viene posta | Meta' fatto |
| 2329 | Errori gestiti senza bloccare l'agente | `InfiniteLoopDetected` **non piu' riproducibile** sulla contabilizzazione servizi | Da confermare |
| 2330 | Distinguere disponibile/rilasciato/da richiedere **e** indicare la procedura | Distinzione presente. **Manca la procedura**: serve sapere da RINA qual e' quella ufficiale | In corso |
| 2331 | Stati di rilascio in positivo | Nessuna doppia negazione in 7 prove | **Done** |
| 2332 | Articoli attivi e corretti per la Legal Entity | Verificato su 3 casi | **Done** |
| 2333 | Una sola risposta per turno | Risolto con l'intervento sull'orchestratore. **Giudizio del mattino ribaltato** | **Done** |
| 2334 | Grassetto solo su intestazioni; niente marcatori senza spiegazione | Grassetto corretto; numerazioni spiegate; resta il marcatore di citazione della piattaforma e il collasso delle liste in F&O | In corso |
| 2335 | Mapping PTEC → nuovi MMD | 4 casi su 4 corretti | **Done** |

### Le due cose che servono da RINA per chiudere

1. **La procedura di richiesta di rilascio articolo** (task 2330). L'agente sa dire *che* un
   articolo va richiesto, ma non *come*. Senza la procedura ufficiale non possiamo scriverla.
2. **La politica sulla lingua** (task 2325). Oggi l'agente segue la lingua dell'utente. Il task
   chiede italiano fisso salvo richiesta esplicita. Inchiodarlo all'italiano e' fattibile in dieci
   minuti, ma penalizza gli utenti non italiani del gruppo: e' una scelta, non un bug.

### Una cosa che decidiamo noi

Il task **2328** chiede la domanda di conferma quando le alternative sono piu' d'una. Oggi l'agente
sceglie da solo. Aggiungere la domanda e' semplice, ma va bilanciato con il task 2324, che chiede
l'opposto: meno domande di ritorno, piu' risultati subito. I due requisiti si toccano e vanno
decisi insieme.
