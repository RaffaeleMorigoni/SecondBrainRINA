---
name: rina-lingua-trascinata-dai-risultati
description: la risposta usciva in italiano solo quando partiva la ricerca articoli; risolto togliendo la prosa italiana dall'output del flusso
metadata:
  type: project
---

Causa radice del "non risponde nella lingua dell'utente" (provata il 15/10/2026 in F&O su RSSE,
stessa chat, due turni consecutivi):

- domanda in portoghese che innesca la **ricerca articoli** -> risposta interamente in italiano
- domanda in portoghese che **non** innesca la ricerca (CAPEX/OPEX, knowledge) -> portoghese perfetto

La regola LANGUAGE funzionava: a romperla era l'**output del flusso**, prosa italiana
("Servizio", "categoria", "unita", "fornitore abituale") che trascinava la lingua.

**Correzione applicata il 15/10 sera** (flusso `RINA - Search Items`, azione `Seleziona articoli`):
l'output non e' piu prosa ma dati strutturati con chiavi inglesi
(`{"code":...,"name":...,"type":...,"category":...,"unit":...,"price":...,"vendor":...}`).
Espressione e rollback in `ROLLBACK_seleziona_articoli_formato.md` (versione 3).

**Esito**: PT e FR ora rispondono nella lingua giusta e riportano la descrizione di anagrafica
verbatim con la traduzione accanto fra parentesi quadre. Non sempre: su 2 prove FR, una e'
ancora uscita in italiano -> il difetto e' **intermittente**, non piu sistematico. Coerente con
"nei miei test 10 min dopo il comportamento e' corretto" riportato dal collega.

**Cosa NON serve** (provato e pubblicato senza effetto, prima di trovare la causa):
- riscrivere la sezione LANGUAGE in forma operativa
- il vincolo esplicito "Tool data is stored in Italian: keep codes and descriptions verbatim,
  write everything else in the user's language" (resta nelle istruzioni padre, 7998/8000)

Collegato a [[rina-formato-fo-non-governabile]], [[copilot-studio-language-rule]],
[[rina-company-sessione-dataareaid]].
