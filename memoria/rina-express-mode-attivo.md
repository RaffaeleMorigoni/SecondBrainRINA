---
name: rina-express-mode-attivo
description: "Dal 21/10/2026 Express mode e' attivo sul flusso RINA - Search Items: taglia ~23s ma consuma capacita'"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T10:27:02.678Z
---

Il 19/10 il toggle **Express mode** sul flusso `RINA - Search Items`
(`f9875f51-7db3-f111-aaac-70a8a537aa21`) sembrava salvare ma tornava Disabled: era un messaggio
di licenza (Process license / capacita' dedicata).

**Il 21/10 si e' attivato davvero.** All'apertura dell'editor compare il banner
*"Based on the run history, this flow qualifies for Express mode"*. Toggle sul **trigger card**
→ `Save draft` → `Publish`. Dopo il salvataggio il toggle **resta ON**: e' il modo per capire se
ha preso.

**Guadagno misurato sullo stesso caso** (manutenzione attrezzature, RSSE, italiano):
`00:01:42` Failed (timeout) → `00:01:16` Succeeded. **~23 secondi.**

**Costo.** Alla pubblicazione il sistema avverte: *"Flow runs consume capacity based on the number
of actions used, but test runs won't consume any."* E' una voce di costo per RINA, da portare in
chiaro prima che qualcuno la scopra in bolletta.

**Non e' la soluzione definitiva.** 1m16s resta vicino al limite che si vede scattare intorno a
1m40s. Il collo di bottiglia vero resta il connettore: vedi
[[rina-search-items-query-sequenziali]] e [[rina-timeout-search-items]].
