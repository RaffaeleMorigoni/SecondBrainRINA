---
name: rina-formato-fo-non-governabile
description: "In F&O la forma delle risposte non segue le istruzioni ed e' instabile; il pannello di test di Copilot Studio non e' rappresentativo"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-08T17:00:54.247Z
---

**Il formato delle risposte non si governa dalle istruzioni, almeno nel canale F&O.** Verificato
l'08/10/2026 con tre domande consecutive nel pannello Copilot dentro F&O, utente su RSSE, subito
dopo aver pubblicato un `ANSWER FORMAT` che chiede intestazioni in grassetto, riga vuota ed
elenco numerato:

| Domanda | Com'e' uscita |
|---|---|
| prestazione professionale 9001 | elenco numerato, intestazioni **senza** grassetto, **piu' un secondo messaggio duplicato** |
| intermediazione immobiliare | **paragrafo unico**, nessuna sezione, nessun elenco |
| manutenzione attrezzature | **paragrafo unico**, nessuna sezione, nessun elenco |

Una risposta su tre ha le due sezioni. Il grassetto non compare mai. Eppure nello screenshot dei
tester dello stesso giorno, con istruzioni che il grassetto **non lo chiedevano affatto**, il
grassetto c'era: il modello lo produce a intermittenza, non perche' glielo si chieda.

**Il pannello di test di Copilot Studio non e' rappresentativo.** Li' la stessa versione produceva
sempre le due sezioni e l'elenco numerato, e in piu' interpreta i titoli markdown `###` che F&O
ignora. Validare la forma nel pannello e' stato un errore: i tester usano F&O, e la maggior parte
delle segnalazioni sulla forma nel registro UAT nasce da li'.

**Come applicarlo:** non spendere altre modifiche alle istruzioni per questioni di impaginazione —
l'ho fatto tre volte di fila senza ottenere il risultato, bruciando lo spazio residuo degli 8.000
caratteri. Se la forma deve essere garantita, va prodotta dal **flusso**, che restituisce gia' i
dati strutturati e puo' consegnare il testo gia' impaginato, togliendo al modello la decisione.
In alternativa va capito se il canale F&O applica un proprio riassunto alle risposte: le risposte
li' sono sistematicamente **piu' brevi e meno strutturate** di quelle del pannello.

Vedi [[rina-template-risposta-ricerca]] per il template che si voleva ottenere e
[[copilot-studio-istruzioni-limite-8000]] per il vincolo di spazio.

**Provato anche dal flusso: non basta.** L'08/10 ho modificato `RINA - Search Items` perche'
consegnasse gli elenchi gia' in markdown — `Items` e `OtherLegalEntities` da
`join(..., '\n')` a `if(empty(...), '', concat('- ', join(..., concat('\n','- '))))`, cioe'
bullet veri invece di righe nude. Pubblicato e provato in F&O:

- "vorrei acquistare un servizio di consulenza informatica" → **formato perfetto**: elenco
  numerato uno per riga, intestazione **in grassetto** con lo stacco, bullet nella seconda sezione;
- "mi trovi il materiale per manutenzione di attrezzature?" → **paragrafo unico**, tutto
  schiacciato, nessun elenco, nessun grassetto.

Quindi nemmeno consegnare il markdown gia' pronto garantisce la forma: il modello riformatta
comunque. **Nota utile**: la risposta ben formattata aveva 6 articoli, quella schiacciata 4. Il
sospetto e' che sotto una certa lunghezza il canale F&O compatti la risposta in prosa. Da
verificare se si vuole insistere.

**Conclusione operativa:** la forma in F&O non e' garantibile ne' dalle istruzioni ne' dal flusso.
Prima di promettere a RINA un formato fisso, va deciso se vale un intervento piu' profondo
(topic dedicato che emette la risposta senza passare dal modello) oppure se si dichiara come
variabilita' nota della piattaforma. La modifica ai bullet nel flusso e' comunque rimasta
pubblicata: non peggiora nulla e migliora i casi in cui la struttura regge.

**08/10/2026 sera - PROVA DECISIVA con "Response formatting".** Esiste in Settings > Generative AI >
Responses un campo **Response formatting** (Preview, 500 caratteri) la cui descrizione dice
testualmente che, in caso di conflitto con le altre istruzioni dell'agente, **queste prevalgono**.
Era vuoto. L'ho compilato con le regole di formato e, per capire se il canale F&O lo riceve davvero,
ci ho messo dentro un marcatore: *inizia ogni risposta con il prefisso "MMD | "*. Salvato,
pubblicato, provato in F&O su RSSE:

| Domanda | Prefisso | Grassetto | Elenco |
|---|---|---|---|
| manutenzione di attrezzature | **presente** | no | no, paragrafo unico |
| consulenza informatica (ieri usciva perfetta) | **presente** | no | no, paragrafo unico |

**Il prefisso arriva, l'impaginazione no.** Letto il DOM: la risposta e' un solo `<span>` dentro un
`<p>`, zero `<strong>`, zero `<ul>/<li>`, zero a capo. Quindi non e' un problema di istruzioni che
non raggiungono il modello - le raggiungono, e su una regola testuale (il prefisso) vengono
rispettate alla lettera. E' la **forma** a essere persa a valle.

**Indizio piu' forte:** la seconda risposta ha perso anche le **descrizioni** degli articoli,
elencando i soli codici `SE707001.0001, SE707002.0001, ...`. Non e' riformattazione: e' perdita di
informazione. Il canale Copilot di F&O applica un proprio passaggio di sintesi sulla risposta.

**Conclusione definitiva:** la forma in F&O non e' governabile da nessuna delle tre leve
disponibili - istruzioni dell'agente, markdown gia' pronto dal flusso, campo Response formatting
che per documentazione dovrebbe sovrascrivere tutto. La dichiarazione di variabilita' nota nel
registro UAT regge. Il campo Response formatting e' rimasto compilato (senza il marcatore di prova)
con in piu' la regola *tieni sempre la descrizione accanto al codice*: non garantisce la forma ma
contrasta la perdita di contenuto appena vista.

---

**08/10/2026, 20:25 — CORREZIONE: con "Response formatting" il formato tiene.** La conclusione
"non governabile" qui sopra vale per le istruzioni e per il markdown dal flusso, **non** per questo
campo. Dopo aver spostato le regole di impaginazione dentro `Response formatting`, tolto il blocco
`ANSWER FORMAT - MANDATORY` dalle istruzioni e ripubblicato, la domanda "vorrei acquistare un
servizio di consulenza informatica" in F&O su RSSE ha prodotto **due volte di fila**:

```
MMD | Articoli disponibili nella tua societa

SE707001.0001 - Consulenza Informatica per applicazioni - Service
... un articolo per riga ...

Articoli presenti in Dynamics ma da richiedere

SE701502.0001 - Service - non rilasciato nella tua societa e da richiedere.
```

con entrambe le intestazioni in `<strong>`, la riga vuota di stacco, un articolo per riga e la
descrizione accanto al codice. **La differenza rispetto ai tentativi falliti e' il campo, non il
testo:** le stesse regole, scritte nelle istruzioni, non venivano applicate; scritte qui, si.

**Perche' le prime prove da questo campo erano uscite in prosa:** perche' il blocco
`ANSWER FORMAT` era ancora nelle istruzioni e i due testi si contendevano la forma. Tolto il
duplicato, il comportamento si e' stabilizzato.

**Regola operativa che ne esce:** la forma va scritta **solo** in `Settings > Generative AI >
Responses > Response formatting` (500 caratteri), mai nelle istruzioni. Le istruzioni restano per
il contenuto. Questo libera anche ~980 caratteri sugli 8.000, che erano il vincolo peggiore.

**Cosa non si e' piegato comunque:** la frase "Vuoi l'elenco completo degli articoli trovati?"
(riga 2 del registro) resiste anche a un divieto esplicito nelle istruzioni, e fra due esecuzioni
identiche il numero di articoli elencati e' passato da 6 a 5 (manca `SE302501.0001`). Il modello
accorcia la lista di suo e poi si offre di completarla. Questo e' il residuo vero da attaccare,
probabilmente dal flusso e non dalle istruzioni.

---

**09/10/2026 - ERRORE DI METODO MIO: le risposte in F&O vanno lette DOPO che lo streaming finisce.**
Ho letto il DOM del pannello Copilot mentre la risposta era ancora in corso, ho visto 44 caratteri
(`MMD | Articoli disponibili nella tua societa`), ho concluso "le risposte si troncano", l'ho
attribuito all'ultima pubblicazione e **ho fatto un rollback che non serviva**. Rileggendo la
stessa risposta piu' tardi era completa: **695 caratteri**, con `MA101021.0001` ed entrambe le
sezioni.

**Come leggere senza sbagliare:** leggere due volte a distanza di ~8 secondi e confrontare; se il
testo e' cambiato, non era finito. Una sola lettura non dice nulla sulla completezza.

```js
const a=read(); await new Promise(r=>setTimeout(r,8000)); const b=read(); // stabile se a===b
```

**Costo dell'errore:** una pubblicazione di rollback inutile e la rimozione temporanea della regola
sul nome della societa', poi rimessa. Nessun danno permanente, ma mezz'ora persa e una diagnosi
sbagliata comunicata a Raffaele.

**Resta vero** che alcune risposte sono genuinamente scarne: su "reagenti chimici" dopo l'ultima
pubblicazione la risposta stabile e' stata una riga sola, `MMD | MA101021.0001 - Fornitura
materiali-prodotti chimici`, senza sezioni e senza nome societa'. La variabilita' della forma
dichiarata alla riga 32 del registro UAT resta il quadro corretto: non inseguirla caso per caso.

## 15/10: il Response formatting governa la FORMA, non gli atti conversazionali

Aggiunta la regola `Chiedi quale codice usare.` nel Response formatting dell'orchestratore
(496/500, pubblicato), per chiudere i task 2324 ridimensionato e 2328. **Non viene applicata**:
due prove su due (consulenza informatica, servizi di pulizia) chiudono suggerendo il codice piu'
adatto, senza mai domandare all'utente quale preferisce.

Le altre regole dello stesso campo funzionano: prefisso `MMD |`, un articolo per riga con la
descrizione accanto, niente paragrafo unico, societa' nominata. Quelle riguardano **come** e'
scritta la risposta.

**Conclusione:** chiedere una conferma e' un atto conversazionale, non di formattazione. Per
ottenerlo serve agire sulle **Instructions** dell'orchestratore, non su questo campo. Stessa
famiglia di [[rina-descrizioni-strumenti-limiti]]: ogni campo governa il proprio livello.

## 15/10: col pochi risultati esce il formato GREZZO del flusso

Verificato in F&O (RSSE) su tre domande prese dai fallimenti della suite. Quando il risultato e'
**uno o due articoli**, l'agente non riformatta: copia la riga come la compone il flusso, campi
tecnici inclusi.

- "Forniscimi il codice materiale per prodotti chimici" ->
  `MMD | MA101021.0001 | Fornitura materiali-prodotti chimici | Goods | category Goods | unit LUMPSUM`
- "MMD per manutenzione degli edifici..." -> due righe grezze con `| category Mainten | unit QTY`

Con molti risultati (laboratorio, consulenza informatica) il formato invece regge. **La soglia
sembra essere il numero di risultati.** Se si vuole chiudere, conviene togliere i campi tecnici
da `Items` nel flusso (`Seleziona_articoli`), non insistere sulle istruzioni.

**RISOLTO il 15/10** riscrivendo l'espressione `select` di `Seleziona_articoli`: la riga ora e'
`CODICE - Descrizione (Servizio|Bene) - categoria X, unita Y` invece del tracciato a pipe con
`category`/`unit` in inglese. Verificato in F&O su tre risposte: niente piu' campi tecnici.
Nessun dato perso. Rollback in `ROLLBACK_seleziona_articoli_formato.md`.

