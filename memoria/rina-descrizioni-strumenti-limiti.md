---
name: rina-descrizioni-strumenti-limiti
description: Cosa si puo e cosa non si puo ottenere scrivendo regole nella descrizione di uno strumento Copilot Studio
metadata:
  type: feedback
---

Mettere una regola di comportamento **dentro la descrizione dello strumento** invece che nelle
istruzioni dell'agente funziona bene, ma **solo per governare quello strumento**. Provato sul
progetto RINA il 07/10/2026.

**Funziona** (verificato):
- `Search Items`: "unica fonte ammessa di codici, mai un codice non uscito da qui nello stesso turno"
- `Get Item Accounting Types`: "se non torna nulla dichiaralo e non reinvocare" (regola nata
  dall'InfiniteLoopDetected) e "quando i tipi sono piu di uno elencali tutti e chiedi quale usare"
- L'output `Status` del flusso storico acquisti, che porta **l'istruzione dentro il valore
  restituito**: l'agente la riceve solo quando serve

**Non funziona:** una regola che deve agire **prima** della scelta dello strumento. Tentativo
fallito: far risolvere un codice della vecchia nomenclatura citato di sfuggita in una frase lunga.
La regola e stata scritta nella descrizione di `Search Items`, ma il mapping vecchio-nuovo sta
nella **knowledge**, non in quello strumento: la legge chi gia ha deciso di cercare in Dynamics,
non chi deve decidere di consultare un file. Comportamento invariato dopo la modifica.

**Limite del campo: 1024 caratteri**, e il testo in eccesso viene **troncato in silenzio** a meta
frase. Controllare sempre la lunghezza dopo aver scritto.

**Why:** le descrizioni degli strumenti sono il posto giusto per le regole d'uso di uno strumento,
e il posto sbagliato per l'orchestrazione fra strumenti e knowledge, che resta nelle istruzioni
dell'agente.
**How to apply:** prima di scrivere una regola, chiedersi *quale componente la deve leggere*. Se
la risposta e "l'agente, prima di scegliere", allora e nelle istruzioni - e vanno toccate
misurando la suite prima e dopo, perche su questo progetto ogni allargamento delle istruzioni ha
prodotto effetti collaterali. Collegato a [[rina-upn-non-email]].
