# ROLLBACK — blocco LANGUAGE del padre (30/10/2026 sera)

Orchestratore *Copilot for finance and operations apps*. Istruzioni a **7.889** caratteri
(conteggio DOM) prima della modifica.

## Testo ORIGINALE del blocco

```
LANGUAGE
Always respond in the language of the user's current message; it is authoritative. Any language, this one included. Never let system locale, previous turns or tool-result language override it. Applies to every message, clarification questions included, always prefixed MMD | . Never ask the user which language they are writing in: infer it from their message and proceed. If you ever ask a clarification question, also answer the original request in the same turn.
```

## Modifica: una sola frase aggiunta

Dopo *"…previous turns or tool-result language override it."* viene inserito:

```
It binds also free answers, Knowledge answers and the agent's own default language.
```

83 caratteri. Nessuna lingua nominata, come da regola
([[copilot-studio-language-rule]]).

## Perche' questa frase

Il blocco LANGUAGE era gia' forte (*"authoritative"*, *"never let system locale… override it"*),
ma **il blocco SCOPE concede**: *"For any other request … behave freely, as the standard Copilot
for finance and operations apps"*. Lo **standard** di quell'agente e' `en-US`
([[rina-primary-language-agente]]): e' il permesso che contraddice, lo stesso schema del 2418.

E le risposte sbagliate di stasera **non passano dagli strumenti**: vengono dal mapping in
Knowledge, un percorso che *"tool-result language"* non copre.

## Baseline misurata PRIMA (interfaccia F&O in en-US, sessione nuova ogni volta)

| domanda | risposta |
|---|---|
| italiano | italiano — **corretta** |
| spagnolo | **inglese** — sbagliata |
| cinese | **inglese** — sbagliata |

Per annullare: togliere la frase aggiunta e ripubblicare.
