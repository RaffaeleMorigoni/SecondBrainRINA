---
name: rina-non-chiedere-la-lingua
description: "L'agente chiedeva all'utente in che lingua scrivesse e poi perdeva la domanda; risolto il 21/10 con una riga nel blocco LANGUAGE del padre"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T11:31:14.988Z
---

**Il sintomo (21/10/2026).** A *"Je dois acheter des réactifs chimiques pour le laboratoire"*
l'agente rispondeva **"Qual è la lingua in cui stai scrivendo il messaggio?"** — in italiano.
E rispondendo *"français"* replicava *"MMD | La langue de votre message est le français."*
**abbandonando la ricerca**.

**Dove NON era.** Sullo strumento `RINA - Search Items` del padre
(`b61db46a-5279-4f67-a415-798a64644b36`), scheda Inputs, **`UserLanguage` non ha l'asterisco
rosso: e' gia' opzionale** (`Keywords` e `UserEmail` ce l'hanno). L'agente non era costretto a
chiedere, **sceglieva** di chiedere. Anche la descrizione dell'input diceva gia'
"never ask the user for it" e non bastava: conferma di
[[rina-descrizioni-strumenti-limiti]] — la descrizione governa lo strumento, non la decisione di
invocarlo. **Quella decisione sta nelle istruzioni del padre.**

**La correzione**, aggiunta in coda al blocco `LANGUAGE` **senza toccare il testo esistente**
(riscrivere quel blocco aveva gia' rotto il francese una volta — vedi
[[rina-lingua-elenco-vs-prosa]]):

```
Never ask the user which language they are writing in: infer it from their message and proceed. If you ever ask a clarification question, also answer the original request in the same turn.
```

Una regola per due difetti: la prima frase impedisce la domanda, la seconda copre il caso in cui
una clarification parta comunque.

Campo da 7.284 a **7.473 / 8.000**, margine 527. Pubblicato 21/10 ore 13:25.

**Verificato** su conversazione pulita, stessa domanda: risposta diretta in francese, nessuna
domanda sulla lingua, un solo messaggio. Italiano non regredito.

**Il metodo che ha funzionato**: prima guardare se l'input era obbligatorio, poi concludere che
era orchestrazione. Senza quel controllo avrei riscritto la descrizione dello strumento per la
seconda volta senza effetto.

## Correzione del 21/10 pomeriggio: regge solo sul primo turno

La verifica di stamattina era su **una conversazione pulita, primo messaggio**. Nel pomeriggio,
testando il 2418, e' ricomparso: domanda in francese -> risposta corretta in francese; poi
**approfondimento nella stessa conversazione** -> *"In quale lingua stai scrivendo il messaggio?"*,
in italiano.

**Quindi la regola non e' sufficiente sul follow-up.** Da verificare sempre su **due turni**, non
su uno: e' il caso segnalato anche da Chiara sul 2418 (PATR), dove la prima risposta e' giusta e
la seconda no.
