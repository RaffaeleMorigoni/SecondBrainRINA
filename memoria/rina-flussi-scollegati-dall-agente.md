---
name: rina-flussi-scollegati-dall-agente
description: I quattro flussi storici non erano piu collegati all'agente in UAT - da chiedere spiegazioni a Chiara
metadata:
  type: project
---

**Scoperto il 07/10/2026.** L'agente `RINA Procurement & MMD Assistant` in UAT aveva **due soli
strumenti**: `MCP Dynamics 365 ERP` (di Chiara DAMBROSIO, un mese fa) e `RINA - Item Purchase
History` (agganciato da noi il 06/10 notte).

**I quattro flussi storici NON erano collegati**, pur esistendo e funzionando in Power Automate:
`RINA - Search Items` (modificato 1 giorno fa), `RINA - Get Category Accounts` (1 settimana),
`RINA - Check Vendor` (2 settimane), `RINA - Get Item Accounting Types` (3 settimane).

**Come si e visto:** suite di regressione del 07/10 alle 09:46, 72% Answer quality, 21/29.
I casi falliti includono *"il codice MA101008.0002 che tipo di contabilizzazione ha e quali conti
movimenta"*, dove l'agente risponde che ha trovato **solo le associazioni codice-descrizione della
knowledge** e non la configurazione contabile: cioe leggeva la knowledge perche la matrice non
era raggiungibile. Falliscono anche le ricerche per parola chiave (`laboratorio`,
`prove di laboratorio`, `prestazione professionale 9001`), che sono il mestiere di `Search Items`.
L'agente se la cavava sugli altri casi passando dall'MCP, che da un accesso generico a F&O.

**Conseguenza sul BRD:** i verdetti "Fase 5, 6 e 7 implementate il 6 ottobre, verifica in corso"
non reggevano: su quell'agente non potevano funzionare.

**DA CHIEDERE A CHIARA** (insieme alla domanda sulle indicazioni AFC): i flussi sono stati
**scollegati di proposito** quando ha introdotto l'MCP - cioe l'MCP doveva sostituirli - oppure si
sono persi in un aggiornamento? La risposta cambia l'architettura: se la scelta era deliberata,
riagganciarli rimette mano a una sua decisione.

**Cronologia accertata:** `Search Items` invocato fino alle **20:43 del 06/10**; quando ho aperto
la scheda Tools verso le **23:00** c'era gia il solo MCP. Sono spariti in quelle due ore e mezza.
Le mie pubblicazioni dell'agente sono **successive**, quindi non possono essere la causa: avevo
gia visto la lista ridotta prima di pubblicare. **Causa ignota**, Copilot Studio non espone uno
storico delle modifiche alla configurazione.

**Ricollegati tutti e cinque il 07/10 (10:30) e agente pubblicato.** Per ognuno:
- descrizione **riscritta a mano** (Copilot Studio mette solo il nome del flusso)
- `UserEmail` = `User.Email` dove l'input esiste: `Search Items`, `Get Category Accounts`,
  `Check Vendor`. `Get Item Accounting Types` prende solo `ItemNumber`, riempito dall'AI
- nelle descrizioni sono state incorporate le regole che faticavano a reggere nelle istruzioni
  dell'agente: `Search Items` unica fonte ammessa di codici e mai un codice non uscito da li nello
  stesso turno; `Get Item Accounting Types` risultato vuoto da dichiarare senza reinvocare (regola
  nata dall'InfiniteLoopDetected); `Get Category Accounts` una categoria non e un attributo
  contabile.

**`RINA - Create RDA Draft` resta scollegato**: e la wave 2, disattivata per scelta di perimetro
(il BRD lo dice gia al capitolo 3).

Collegato a [[rina-flusso-storico-acquisti]].

**Why:** senza questi strumenti meta della suite misura un agente monco, e i verdetti del BRD
sono falsati.
**How to apply:** prima di fidarsi di un punteggio della suite, controllare che la scheda Tools
dell'agente contenga tutti e cinque i flussi. Copilot Studio **non eredita la descrizione del
flusso**: ogni riaggancio richiede di riscriverla a mano, altrimenti l'orchestratore non sa quando
usare lo strumento.
