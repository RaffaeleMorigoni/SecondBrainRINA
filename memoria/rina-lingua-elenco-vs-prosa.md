---
name: rina-lingua-elenco-vs-prosa
description: la risposta esce in italiano quando l'agente fa un elenco puntato di codici, nella lingua giusta quando scrive in prosa
metadata:
  type: project
---

Regola trovata il 15/10/2026 con 9 prove in F&O su RSSE, formulazione identica e solo
l'argomento variabile:

**elenco puntato di codici -> risposta in italiano. Prosa continua -> lingua dell'utente.**

| argomento | lingua | forma | esito |
|---|---|---|---|
| consulenza informatica | PT | prosa | PT ok |
| consulenza informatica | FR (x2) | prosa | FR ok |
| reagenti chimici (chiarimento) | FR | prosa | FR ok |
| manutenzione attrezzature | FR (x3) | elenco | italiano |
| microscopi elettronici | FR | elenco | italiano |
| formazione personale | FR | elenco | italiano |

Quando il modello compone un elenco incolla le descrizioni di anagrafica riga per riga e resta
in italiano; quando scrive in prosa le integra nella frase e le traduce, mettendo l'originale
fra parentesi quadre. Prove in `plans/prove_uat/20261015_FR_*`.

**Ipotesi scartate, con prova:**
- non e' la formulazione della domanda: stesso argomento con francese marcato e accenti -> italiano
- non e' il volume dei risultati: microscopi elettronici ha 4 risultati e fallisce, manutenzione
  ne ha 33 e fallisce, consulenza ne ha ~5 e riesce
- non e' intermittente: ogni argomento si comporta sempre allo stesso modo

**Conseguenza**: l'elenco e' richiesto dal nostro Response formatting ("one item per line as a
bullet, description next to the code"), quindi e' il template a innescare il difetto.
Collegato a [[rina-lingua-trascinata-dai-risultati]] e [[rina-template-risposta-ricerca]].

## Tentativo sul template (16/10/2026, pubblicato 00:43) - esito parziale

Response formatting dell'orchestratore, prima frase sostituita (490/500, rollback in
`ROLLBACK_response_formatting_16ott.md`):
`Answer wholly in the user's language, descriptions too: translate each, stored text in brackets.`

Esito su 4 prove:
- microscopi elettronici FR: **francese** (prima italiano) -> ma la risposta e' diventata prosa
- manutenzione attrezzature FR: italiano, elenco -> invariato
- formazione personale FR: italiano, elenco -> invariato
- manutenzione attrezzature IT: elenco uno per riga, nomina RSSE -> **nessuna regressione**

Conferma la regola: la frase sulla traduzione funziona solo dove la risposta esce in prosa.
Dentro l'elenco puntato il modello continua a incollare la descrizione italiana.
La modifica resta pubblicata perche' migliora un caso e non peggiora nulla.

## 21/10/2026 - la causa del formato "originale [traduzione]" e' nel prompt del FIGLIO

Nel blocco LANGUAGE OVERRIDE delle istruzioni del connected agent
(`agents/b4a1f29d-c9f8-4c8d-889c-f1906ea65b5f`) c'e' questa regola, **preesistente**:

> "Item descriptions, category names and accounting type descriptions come from Dynamics in their
> original language, **usually English**: always show that original text **unchanged** so the user
> can match it in F&O, and when the user writes in another language **add a short translation in
> brackets** right after it, for example "Purchase of Hdw - Pcs (CAPEX) [acquisto hardware PC]".
> Never replace the original text with the translation."

**E' la causa diretta del formato segnalato nel 2418** ("descrizione italiana + traduzione fra
parentesi"), non il solo instradamento. La regola era sensata quando le descrizioni arrivavano
non tradotte; da quando il flusso restituisce ProductTranslations nella lingua dell'utente, per
il percorso che passa dal flusso e' **obsoleta e dannosa**.

Perche' non si vede sempre: se la descrizione arriva gia' nella lingua dell'utente (percorso
flusso) non c'e' nulla da aggiungere fra parentesi. Si vede quando la descrizione arriva da
un'altra fonte in italiano o inglese (MCP, knowledge, o nessuna chiamata al flusso).

**Attenzione prima di toccarla**: il 16/10 avevo gia' provato a riscrivere la frase
"usually English" e **aveva rotto il francese**; era stata ripristinata al testo esatto. Non
riscriverla a mano libera: va cambiata solo insieme a una prova sui tre casi francesi.
