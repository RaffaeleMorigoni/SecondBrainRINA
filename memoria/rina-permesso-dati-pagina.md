---
name: rina-permesso-dati-pagina
description: "La mancata invocazione della ricerca non era un divieto mancante ma un permesso di troppo: il ramo libero delle istruzioni autorizzava a rispondere dai dati della pagina aperta"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-28T16:58:17.037Z
---

**Il blocco SCOPE dell'orchestratore ha due rami**: le richieste Procurement/MMD/RDA vanno instradate,
*"for any other request (general Dynamics 365 how-to, questions about the app data or pages, …)
behave freely… answer normally with… the app data you can access"*.

**Quel secondo ramo era la causa del 2418.** Una domanda come *"item number EX102002.0001"*, posta
mentre quella riga e' a video in *Released product details*, e' classificabile come domanda sui dati
della pagina: l'agente prendeva il ramo libero, restituiva il `Product name` della griglia — sempre
in italiano — e non invocava mai `RINA - Search Items`. Non stava violando nulla.

**La lezione generale: prima di aggiungere un divieto, cercare il permesso che lo contraddice.**
Il 21/10 ho aggiunto tre regole proibitive, nessuna ha retto a freddo, perche' il modello non stava
disobbedendo. Il 28/10 ho tolto il permesso (`…or pages other than item, product or MMD codes,`)
e unificato la regola sulle fonti, estendendo alla pagina quello che valeva per la Knowledge
(vedi [[rina-knowledge-compete-con-la-ricerca]]: funziona **dire a cosa serve una fonte**, non
vietarla in astratto).

**Come si prova:** solo in F&O, con una riga prodotto selezionata a video, domanda in lingua diversa
dall'italiano. Il criterio e' **se parte la run del flusso**, non com'e' scritta la risposta. E va
ripetuto a freddo il giorno dopo ([[rina-test-regressione-manutenzione]]).

Dettaglio in `plans/20261028_fix_mancata_invocazione.md`.

## 29/10/2026 - PROVA A FREDDO SUPERATA

In F&O, RSSE, riga `EX102002.0001` a video, interfaccia italiana, domanda in inglese
(`item number EX102002.0001`), il giorno dopo la pubblicazione: **la run del flusso e' partita**
(`Oct 29, 02:27 PM`). Prima il Copilot rispondeva subito col `Product name` della griglia e
**nessuna run**. Il 2418 e' chiuso sul suo difetto: togliere il permesso ha funzionato dove tre
divieti non avevano retto.

**Ma la run e' finita in `TimedOut` a `00:02:00`** e l'utente ha visto `FlowActionTimedOut`.
Il difetto si e' spostato dal routing al 2494. Vedi
[[rina-ricerca-parole-italiano-iniziaper]] e `plans/20261029_2418_afreddo_e_collo_di_bottiglia.md`.
