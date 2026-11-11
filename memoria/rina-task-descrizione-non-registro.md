---
name: rina-task-descrizione-non-registro
description: Giudicare un task DevOps sulla sua descrizione, non sulla riga del registro UAT da cui nasce
metadata:
  type: feedback
---

I task DevOps della PBI 1812 nascono dalle righe del registro UAT, ma **non dicono la
stessa cosa**. La riga e' lo sfogo del tester; la descrizione e' il requisito concordato,
spesso piu' stretto o piu' largo.

Il 09/10/2026 tre giudizi su nove erano sbagliati per questo:
- **2327** — letto come "l'agente deve conoscere le deleghe" (impossibile, manca il dato).
  La descrizione chiedeva di *guidare l'utente a verificarle in F&O*: gia' soddisfatto.
- **2330** — scritto "non affrontato". La distinzione disponibile/da richiedere c'era gia';
  mancava solo la seconda meta' ("indicando la procedura corretta").
- **2333** — scritto "architetturale, non risolvibile": era gia' risolto dall'intervento
  sull'orchestratore.

**Why:** l'utente ha corretto con "non capisco cosa non ti torna io credo funzioni" su
2332, e aveva ragione. Il giudizio severo o indulgente nasce sempre dal leggere la fonte
sbagliata.

**How to apply:** aprire il task, leggere la **Description** per intero, contare quante
cose chiede (spesso sono due unite da "e"), e verificarle una per una in F&O prima di
scrivere il commento. Vedi [[rina-suite-regressione]].
