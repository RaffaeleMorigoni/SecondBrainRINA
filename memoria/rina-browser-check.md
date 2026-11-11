---
name: rina-browser-check
description: "Prima di usare Chrome per RINA verificare che il browser collegato sia quello con la sessione RINA, non quello di un altro cliente"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-09-17T10:19:02.381Z
---

Il 17/09/2026 l'unico Chrome collegato ("Browser 1", Windows, questa macchina) era la sessione di un altro cliente: Copilot Studio rimandava a "environments/~personal" invece che a rina-uat-clm. L'utente ha detto "non è quella macchina da usare" / "sei sempre sulla sessione aperta di un altro cliente".

Il Chrome giusto per RINA risultava deviceId 0e81ed0a-08aa-4581-af24-6b8b441c71fe; b02b1763-b88b-485d-82d2-632c5a455136 era quello dell'altro cliente (gli ID possono cambiare: verificare sempre l'ambiente).

**Why:** agire nel browser sbagliato espone o modifica dati di un altro cliente.
**How to apply:** all'inizio di ogni lavoro RINA via browser, chiamare list_connected_browsers e verificare l'URL/ambiente (rina-uat-clm, utente C_110912@rina.org) prima di qualsiasi azione; se l'ambiente non torna, fermarsi, chiudere le schede aperte e chiedere all'utente di collegare il Chrome corretto. Collegato a [[copilot-studio-browser-quirks]].
