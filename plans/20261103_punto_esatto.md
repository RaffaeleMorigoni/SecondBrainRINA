# Punto esatto — 03/11/2026

**Data dell'ultima misura: venerdi' 30/10, sera.** Da allora **nessuna prova**: sono passati
quattro giorni e le quattro modifiche pubblicate quella sera **non sono mai state riverificate**.
La regola del 21/10 dice che una prova riuscita subito dopo la pubblicazione puo' sciogliersi
nella notte: finche' non si riprova, quanto segue e' **lo stato al 30/10**, non lo stato di oggi.

---

## A. Pubblicato e in piedi (4 modifiche, tutte del 30/10)

| | cosa | misurato allora |
|---|---|---|
| **Modello** | padre e standalone su **GPT Chat Latest** (prima del ritiro di GPT-5.5 del 30/10) | formato corretto in F&O, risposte piu' asciutte |
| **Keywords** (flusso) | la descrizione impone di tradurre le parole chiave **in italiano** prima della query | da domanda portoghese sono arrivate `certificazione, ISO, servizio` |
| **Lingua dei campi** (figlio `b4a1f29d`) | tolta la premessa falsa *"usually English"* | caso Emanuela: `Cout OPEX direct [Costo OPEX diretto]` |
| **Blocco LANGUAGE** (padre `679d8694`) | esteso a risposte libere e da Knowledge, +83 caratteri | **ES e ZH** da inglese a corretti, **nessuna regressione** |

Istruzioni padre: **7.973 / 8.000** (27 caratteri liberi).
Istruzioni figlio: **29.550**.

**Ripristinato lo stesso giorno**: il *contiene* sulle parole corte nelle query — raddoppiava i
tempi e non era mai stato validato. Le query sono **esattamente com'erano**.

## B. Stato sulle lingue al 30/10 (interfaccia F&O in inglese = caso peggiore)

| lingua | esito |
|---|---|
| italiano, spagnolo, cinese | **corrette** |
| portoghese | lingua corretta, **non comunica il codice** |
| francese | **inglese** |

Il francese **non e' una regressione**: provato anche **senza** la modifica, risponde in inglese
lo stesso. Difetto preesistente, mai emerso perche' nessuna segnalazione dichiarava la lingua di
interfaccia di chi provava.

## C. Difetti aperti, in ordine di gravita'

1. **Codici che non vengono dalla ricerca** — la ricerca aveva restituito cinque codici, la
   risposta ne conteneva **sei diversi**. Presi da Knowledge o dalla memoria del modello.
   Il formato e' impeccabile, quindi **non si vede**. Finche' e' aperto, nessun test letto a
   video e' attendibile.
2. **Le keyword cambiano a ogni esecuzione** — stessa domanda, venti minuti dopo, parole diverse.
   Spiega l'intermittenza: la ricerca trova o no a seconda del sorteggio.
3. **Due variabili dell'orchestratore prendono la lingua dal locale F&O** — contraddicono il
   blocco LANGUAGE. Non toccate: sono dell'agente Microsoft in soluzione gestita.
4. **Francese in inglese** (vedi B).
5. **2494**: query lente, `'*parola*'` con wildcard in testa.
6. **Portoghese**: trova il mapping e non lo comunica, chiede validazione in Dynamics.

## D. Suite da 50 casi (30/10)

Due run: **68/42** e **64/36** (storico 76-82 / 52-64). Ma **otto fallimenti su dodici sono il
fallback generico**, sugli **stessi otto casi** in entrambe le run, e in F&O quelle domande
rispondono correttamente. **I numeri non sono confrontabili con lo storico** finche' quegli otto
restano.

---

## Cosa manca per avere un punto davvero "di oggi"

Le prove a freddo, che richiedono l'accesso a F&O:

1. **caso di regressione** — manutenzione attrezzature in italiano;
2. **caso Emanuela** — RDA ISO 9001 in francese, per la lingua dei campi;
3. **spagnolo con interfaccia en-US** — la modifica piu' recente;
4. **controllo di provenienza** su uno dei codici in risposta, confrontandolo con gli output del
   flusso in Power Automate (regola nuova del 30/10).

I primi tre sono ~10 minuti. Il quarto e' quello che conta di piu'.

## Da chiedere

- **Chiara**: chi ha configurato le **nove lingue secondarie** dell'orchestratore il 20/10 alle
  20:19 e a cosa servono. Non le abbiamo messe noi.
- **Tester**: ogni segnalazione sulla lingua deve dichiarare la **lingua di interfaccia F&O**.
  Senza quel dato, il 30/10 ho dato due diagnosi sbagliate.
