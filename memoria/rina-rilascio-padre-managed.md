---
name: rina-rilascio-padre-managed
description: "L orchestratore F&O e un componente managed di Microsoft con le nostre istruzioni in un unmanaged layer: non si pacchettizza nella solution"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-11-11T08:53:12.133Z
---

Verificato in UAT l 11/11/2026, in `make.powerapps` → solution **Dynamics 365 F&O Procurement Expert** (`fe0a1b50-947e-f111-ab0e-70a8a537aa21`, unmanaged, publisher HUDi).

L agente **Copilot for finance and operations apps** (`679d8694-1250-4780-946e-6db1c2109ae1`) risulta `Managed = Yes`, `Customized = Yes`, e i suoi solution layer sono due: `msdyn_FnOCopilot` (Dynamics 365) alla base, e sopra un **Unmanaged layer Active** del Default Publisher. Le nostre istruzioni vivono li. Il figlio `RINA Procurement & MMD Assistant` ha invece **un solo layer unmanaged**: componente pulito, deployabile.

Nella solution sono finiti anche componenti di Microsoft: due connection reference con prefisso `msdyn_fnocopilot` e due environment variable estranee (System Optimization Dashboard API config, QEA Fabric MCP Server URL), entrambe vuote.

Nessuna pipeline configurata. Vincolo di piattaforma: i target di una pipeline devono essere **Managed Environments** — questione di licenza, da verificare su produzione.

**Why:** avevo impostato il piano di rilascio come export-import. Non lo e: piu della meta della configurazione (istruzioni del padre, collegamento padre-figlio, knowledge, modello, Express mode, condivisioni) non viaggia in una solution, e un unmanaged layer in produzione **batte** qualunque managed importata, facendo risultare verde una pipeline che non ha cambiato nulla.

**How to apply:** nella solution va solo roba nostra — figlio, flussi, connection reference nostre. Le istruzioni del padre si trattano come configurazione d ambiente, versionata come testo in `plans/` e applicata a mano con checklist: e anche l unico rollback possibile per quella parte. Resta da accertare, aprendo lo zip dell export 1.0.0.2, se l export catturi o no le istruzioni del padre. Dettaglio in [[rina-suite-regressione]] per i test post-deploy e nei file `plans/20261111_*`.
