---
name: rina-blocco-language-ambito
description: "La regola sulla lingua non copriva le risposte libere e quelle da Knowledge: una frase di 83 caratteri ha risolto spagnolo e cinese, misurato prima e dopo"
metadata:
  type: project
---

**30/10/2026, pubblicato alle 18:30.** Nel blocco `LANGUAGE` del padre, dopo *"Never let system
locale, previous turns or tool-result language override it."*:

```
It binds also free answers, Knowledge answers and the agent's own default language.
```

**Misurato prima e dopo**, interfaccia F&O in `en-US` (la condizione che fa uscire il difetto),
sessione nuova ogni volta, stessa domanda in tre lingue:

| domanda | prima | dopo |
|---|---|---|
| italiano | italiano | **italiano** (nessuna regressione) |
| spagnolo | **inglese** | **spagnolo** |
| cinese | **inglese** | **cinese** |

**Perche' mancava**: il blocco era gia' forte, ma due percorsi gli sfuggivano - il ramo libero di
SCOPE (*"behave freely, as the standard Copilot for finance and operations apps"*, e lo standard
e' `en-US`, [[rina-primary-language-agente]]) e le risposte dalla **Knowledge**, che non sono
"tool-result" e coprono tutte le domande sui codici legacy.

**Terza volta in un giorno** che la causa e' un **permesso di troppo** e non un divieto mancante,
dopo [[rina-permesso-dati-pagina]] e [[rina-lingua-campi-dynamics]]. La frase non vieta: estende
l'ambito di una regola che c'era gia'.

Istruzioni da 7.889 a **7.973 / 8.000**: restano 27 caratteri, verificata la coda contro il
troncamento silenzioso ([[copilot-studio-istruzioni-limite-8000]]).

**Portoghese e francese, provati subito dopo**: portoghese **corretto nella lingua** ma non da'
il codice (*"o material atual ainda precisa ser validado no Dynamics"*) - e' contenuto, non lingua.
**Francese: inglese, due prove su due** - unica lingua ancora rotta. Non e' intermittenza, e non
posso dire se sia regressione perche' la baseline francese con interfaccia `en-US` non esiste.

**Bilancio: 4 lingue su 5 corrette nella lingua** (IT, ES, ZH, PT), il francese resta aperto.

**A/B fatto sul francese**: tolta la frase e ripubblicato, il francese **risponde in inglese lo
stesso**. Non e' una regressione: e' un difetto **preesistente**, mai misurato prima in questa
condizione. La frase e' stata **rimessa** perche' migliora ES/ZH e sul francese e' neutra.

**Il francese resta l'unico caso aperto** ed e' anomalo: unica lingua che fallisce, e in modo
stabile. Da escludere qualcosa di specifico - fra le lingue secondarie configurate il 20/10 da
altri compare `French (France) (fr-FR)` ([[rina-primary-language-agente]]). Dettaglio in
`plans/20261030_fix_blocco_LANGUAGE.md`, ripristino in
`plans/backup/20261030_ROLLBACK_padre_blocco_LANGUAGE.md`.
