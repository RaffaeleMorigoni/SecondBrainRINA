---
name: rina-monitoring-sessioni-dove-guardare
description: "Per sapere chi ha usato l'agente serve Analytics > Sessions, non Activity: quella vista e' incompleta e porta a dire che non ha testato nessuno"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-28T15:42:49.216Z
---

**La fonte giusta e' `Monitor → Analytics → Sessions`**
(`…/bots/<id>/analytics/sessions`): filtro per periodo, durata, numero di messaggi, esito
(Resolved / Unengaged / Abandoned) e **trascrizione completa** di ogni sessione.

**`Activity` NON va usata per questa domanda.** Il 28/10 mostrava come piu' recente il 22/10, e
tutte le righe a mio nome: stavo per rispondere a Giorgio che **non aveva testato nessuno**.
Analytics sulla stessa finestra mostrava **6 sessioni, 3 delle quali quel giorno stesso**.

**L'esito dichiarato non e' affidabile.** Una sessione del 28/10 risulta **Resolved** ma la
trascrizione mostra l'agente che chiede tre volte di connettere le credenziali, senza mai
rispondere. **Aprire sempre la trascrizione** prima di riportare un dato aggregato.

**Cosa si scopre solo di li':** i blocchi che l'utente incontra *prima* di qualunque risposta —
`You don't have access to talk to this bot`, la card *Open connection manager*, i
`FlowActionTimedOut`. Sono invisibili sia in Activity sia nelle run di Power Automate (se il
flusso non parte, non c'e' nessuna run da guardare).

Vedi `plans/20261028_sessioni_utenti_RINA.md`.
