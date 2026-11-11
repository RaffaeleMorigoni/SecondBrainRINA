---
name: rina-dlp-knowledge-bloccata
description: "Il blocco DLP sulle knowledge file RINA e' rientrato; dal 07/10/2026 sono Ready e i fallimenti da codice vecchio hanno un'altra causa"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-07T13:44:09.576Z
---

**Il blocco e' rientrato.** Al 07/10/2026 le due knowledge file dell'agente
(`Old_new_only_RINA (1).xlsx` e `RINA_MD_Material_Codes_v1 (2).xlsx`) sono **Ready**,
con "Refresh successful". Nessun `DataLossPreventionViolation`.

Storico: il 30/09/2026 risultavano **Blocked**, tooltip "Not allowed due to your
organization's data loss prevention policies"; il 29/09 alle 16:08 funzionavano ancora.
Non era un problema di permessi ne' di security group: il DLP classifica i connettori,
non gli utenti, e colpiva anche il maker. **Da verificare in produzione prima di
rilasciare**, perche' nessuno ha spiegato ne' il blocco ne' il rientro.

**Attenzione a non riusare questa come spiegazione.** Il 07/10 ho attribuito a questo
blocco i fallimenti sui codici vecchi (`Npprom141`, `Npinfo521`) nella suite, e mi sono
sbagliato. La traccia di esecuzione mostra la causa vera: l'agente chiama
`RINA - Search Items` passando il codice vecchio come `Keywords`, lo cerca nel catalogo
F&O dove per definizione non esiste, e **non consulta mai la knowledge**. Vedi
[[rina-knowledge-mapping-oldnew]]: il mapping e' knowledge, non somiglianza.

**Come applicarlo:** prima di dire che una domanda fallisce per la knowledge, aprire la
traccia nel pannello di test e guardare quali nodi sono stati eseguiti davvero. Lo stato
delle knowledge si legge in un secondo nella scheda Knowledge. Vedi [[rina-suite-regressione]].
