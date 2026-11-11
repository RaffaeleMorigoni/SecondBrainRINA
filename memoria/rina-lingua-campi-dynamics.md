---
name: rina-lingua-campi-dynamics
description: "I campi da Dynamics uscivano in italiano perche' le istruzioni dicevano che arrivano 'usually English': corretto il 30/10, ora lingua utente con l'italiano fra parentesi"
metadata:
  type: project
---

**Segnalazione Emanuela, 30/10/2026**: *"continuano ad esserci problemi con la lingua"*. La prosa
era francese corretta; a uscire in italiano erano i **campi che arrivano da Dynamics**:
`2 - D - Costo OPEX diretto [cout OPEX direct]`.

**Causa: una premessa falsa nelle istruzioni del figlio.**

> *…come from Dynamics in their original language, **usually English**: always show that original
> text unchanged so the user can match it in F&O…*

Le descrizioni arrivano **in italiano**. Da quella falsita' discendeva l'ordine di stampare
l'italiano. L'agente obbediva - stesso schema di [[rina-permesso-dati-pagina]] e delle keyword.
E la ragione addotta non regge: con [[rina-traduzioni-prodotto-esistono]] un francese in F&O
vede il francese.

**Serviti due passaggi.** Nel primo avevo scritto *"metti l'italiano fra parentesi solo quando non
c'e' traduzione"*: i tipi di contabilizzazione e i nomi dei conti, che traduzioni non ne hanno,
sono usciti in **italiano nudo** - peggio di prima. La regola era rovesciata. Versione buona:

> *…when Dynamics has no translation for a field, such as accounting type descriptions and account
> names, **translate the Italian text yourself** into the user's language and put the Italian
> original in brackets right after it.*

**Regola generale che ne esce**: una regola sulla lingua deve dire **cosa fare quando la
traduzione non c'e'**, altrimenti il modello ripiega sull'originale. I campi si dividono in tre:
tradotti da Dynamics, da tradurre a mano (contabilizzazione, conti), **mai da tradurre** (codici,
`External`, `Accrfees`, `LUMPSUM`).

Verificato in F&O sulla domanda di Emanuela. Istruzioni figlio da 29.447 a 29.550 caratteri,
pubblicato alle 17:13. Ripristino in `plans/backup/20261030_ROLLBACK_figlio_riga_lingua.md`,
dettaglio in `plans/20261030_fix_lingua_descrizioni.md`.
