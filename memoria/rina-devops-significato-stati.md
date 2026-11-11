---
name: rina-devops-significato-stati
description: "Su DevOps RINA 'Ready for testing' vuol dire testato da noi e pronto per il cliente, non in attesa di un nostro test"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T13:29:38.102Z
---

**Ready for testing = verificato da HUDi, pronto da consegnare.** Non significa "qualcuno di noi
deve ancora testarlo".

Il flusso e':

1. `To Do` / `In Progress` — lavorazione nostra.
2. **`Ready for testing`** — **noi abbiamo finito e testato**. Al **SAL serale** il team funzionale
   comunica al cliente che e' rilasciato da noi.
3. `Done` — gli **utenti di business** di RINA l'hanno provato e validato.

**Perche' conta.** Il 21/10 avevo letto tre task in Ready for testing (2404, 2295, 2424) come
"fermi, nessuno li prende" e li avevo segnalati come collo di bottiglia: **sbagliato**, erano
pronti per il cliente. E avevo portato il **2422 a Done** dopo averlo testato io: anche quello
sbagliato, perche' Done spetta al business. Riportato a Ready for testing lo stesso giorno.

**Regola operativa:** quando finisco e verifico qualcosa, lo porto a **Ready for testing**, mai a
Done. Done lo mette chi raccoglie l'esito del business.

**Corollario sul SAL serale:** quello che passa a Ready for testing durante la giornata e' esatta-
mente la lista che il team funzionale comunica la sera. Vale la pena tenerla pronta e ordinata.
