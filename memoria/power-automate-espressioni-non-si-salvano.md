---
name: power-automate-espressioni-non-si-salvano
description: Nel designer Power Automate l'editor fx non committa: scrivere l'espressione a mano nel campo con la @ iniziale
metadata:
  type: feedback
---

Nel designer di Power Automate (ambiente RINA, flussi agente) il popup dell'editor di espressione **non committa mai**: si digita, si clicca Update/Add, il popup si chiude e il valore torna quello di prima. Succede sia nel designer nuovo sia in quello classico, sia su campi esistenti sia su azioni appena create, e Copilot del designer si rifiuta di scrivere.

Via che funziona: svuotare il campo del parametro (clic nel campo, Ctrl+A, Canc) e **digitare l'espressione direttamente nel campo con la `@` iniziale**, per esempio `@concat('ItemNumber eq ', ...)`. Il valore finisce nella definizione: si verifica subito nella scheda **Code view** dell'azione, che è live.

Il campo resta segnato "Invalid parameters / Enter a valid JSON" e **Publish viene bloccato dal Flow checker**. Non è un errore reale: basta **Save draft, ricaricare la pagina** — il designer rilegge la definizione e riconverte il testo in token, l'errore sparisce — e poi Publish.

**Why:** senza questo giro il lavoro sembra fatto e invece viene scartato in silenzio; è successo per giorni sul difetto `unterminated string literal` di [[rina-suite-regressione]].
**How to apply:** ogni modifica di espressione in questi flussi va fatta così e verificata in Code view prima e dopo la pubblicazione, non dandosi per buoni i clic sull'editor fx.

**Caso a parte — i campi di una Condition.** Lì il designer avvolge *sempre* il valore in `@{...}`, quindi il risultato è una **stringa**, anche scrivendo `@empty(...)` senza graffe o passando dall'editor fx. Se il lato destro è `true` o un numero, il designer lo salva come booleano/numero e il confronto è **sempre falso**: il ramo True non parte mai e non c'è nessun errore a segnalarlo.
Via che funziona: confrontare **stringa con stringa**, per esempio sinistra `@{concat('X', trim(coalesce(triggerBody()?['text_2'],'')))}` e destra `X`. Verificare sempre in Code view che i due lati siano entrambi fra virgolette.
