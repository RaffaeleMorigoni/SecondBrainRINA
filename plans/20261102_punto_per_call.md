# Punto della situazione — 02/11/2026

**Avvertenza**: tutto quello che segue e' stato pubblicato **venerdi' 30/10 sera**.
**Nessuna modifica e' stata riprovata a freddo**: oggi e' il primo giorno utile, e la regola
(21/10) dice che una prova riuscita subito dopo la pubblicazione puo' sciogliersi nella notte.

---

## 1. Fatto e pubblicato (4 modifiche)

| | cosa | esito misurato |
|---|---|---|
| **Modello** | padre e standalone da GPT-5.5 a **GPT Chat Latest**, prima del ritiro del 30/10 | formato corretto in F&O; risposte piu' asciutte |
| **Keywords** | la descrizione ora impone di tradurre le parole chiave **in italiano** prima della query | funziona: da domanda portoghese sono arrivate `certificazione, ISO, servizio` |
| **Lingua dei campi** (figlio) | tolta la premessa falsa *"usually English"*: ora lingua utente con l'italiano fra parentesi | verificato sul caso di Emanuela: `Coût OPEX direct [Costo OPEX diretto]` |
| **Blocco LANGUAGE** (padre) | esteso alle risposte libere e a quelle da Knowledge | **spagnolo e cinese** da inglese a corretti, nessuna regressione |

**Ripristinato e non lasciato in piedi**: il *contiene* sulle parole corte nelle query — costava
il doppio e non era validato.

## 2. Lo stato sulle lingue (interfaccia F&O in inglese, il caso peggiore)

| lingua | risposta |
|---|---|
| italiano, spagnolo, cinese | corrette |
| portoghese | lingua corretta, **non comunica il codice** |
| **francese** | **inglese — aperto** |

Il francese non e' una regressione: provato anche **senza** la modifica, risponde in inglese lo
stesso. E' un difetto preesistente mai emerso prima perche' **nessuna segnalazione dichiarava la
lingua di interfaccia** di chi provava.

## 3. I due difetti piu' seri, entrambi aperti

**A. L'agente risponde con codici che non vengono dalla ricerca.**
Confrontando la risposta con gli output del flusso: la ricerca aveva restituito cinque codici,
la risposta all'utente ne conteneva sei **completamente diversi**. Presi da Knowledge o dalla
memoria del modello. Il formato e' impeccabile, quindi **non si vede**. E' il piu' grave perche'
produce risposte sbagliate che sembrano giuste.

**B. Le parole chiave cambiano a ogni esecuzione.**
Stessa domanda, venti minuti dopo, parole diverse. Spiega l'intermittenza inseguita tutto il
pomeriggio: la ricerca trova o non trova a seconda di quali parole sono state estratte.

## 4. Una causa strutturale trovata venerdi' sera, non ancora toccata

Due variabili globali dell'orchestratore dicono che **la lingua si prende dal contesto F&O**,
cioe' dal locale:

> *"Lingua corrente dell'utente acquisita dal contesto di Dynamics 365 F&O. Utilizzata per passare
> al RINA Procurement & MMD Assistant la lingua da utilizzare nella risposta."*

Contraddice il blocco LANGUAGE e la regola vera (la lingua la da' la domanda). **Quarto
permesso-che-contraddice** trovato in un giorno. Non toccate: sono dell'agente Microsoft in
soluzione gestita, e un errore li' si propaga a tutte le lingue.

## 5. Suite da 50 casi

Due run venerdi': **68/42** e **64/36** (storico 76-82 / 52-64). Ma **otto fallimenti su dodici
sono il fallback generico**, gli stessi otto casi in entrambe le run, e in F&O quelle domande
rispondono. I numeri non sono confrontabili con lo storico finche' quegli otto restano.

---

## Prossimi passi, in ordine

1. **Prova a freddo oggi** delle quattro modifiche: e' la prima cosa, prima di aggiungere altro.
2. **Chiudere i codici fuori dalla ricerca** — viene prima di tutto il resto, perche' finche' e'
   aperto nessun test basato sulla risposta a video e' attendibile.
3. **Guardare le due variabili lingua** in *Variables*: chi le valorizza, chi le legge.
4. **L'estrazione non deterministica** delle keyword: finche' resta, ogni misura e' un sorteggio.
5. **Il 2494** (query lente) dopo i punti 2 e 4, non prima.

## Da chiedere in call

- **Chiara**: chi ha configurato le **nove lingue secondarie** dell'orchestratore il 20/10 alle
  20:19 e a cosa servono — non le abbiamo messe noi.
- **Ai tester**: ogni segnalazione sulla lingua deve dichiarare la **lingua di interfaccia F&O**
  di chi prova. Senza quel dato i casi non sono confrontabili, e venerdi' mi e' costato due
  diagnosi sbagliate.
