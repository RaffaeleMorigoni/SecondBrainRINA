---
name: copilot-studio-language-rule
description: "User wants agent/tool language rules generic (user's language, any), never a hardcoded list of languages"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-09-09T14:53:03.157Z
---

For Copilot Studio agents/tools (RINA project), language handling must be a generic rule: "always answer/ask in the user's language (UserLanguage / language of the last message), whatever it is". Never hardcode examples like "Italian, Portuguese, English".

**Why:** RINA users can write in any language (a Brazilian in Portuguese, Chinese, ...). Hardcoded examples bias the model: the child agent listed only IT/EN and fell back to English for Chinese until a generic LANGUAGE OVERRIDE was added.

**How to apply:** In tool input descriptions and agent instructions write the generic rule; test with at least one non-listed language (e.g. Chinese) before declaring language handling done. See [[copilot-studio-browser-quirks]].

**Deciso il 12/10/2026: la lingua si prende dal TESTO della domanda, non da F&O.**
Esiste `Global.PA_Copilot_ServerForm_UserContext.userLanguageId` (la lingua delle User
options, vedi [[rina-company-sessione-dataareaid]]) e sarebbe stato un modo per smettere di
dedurla. Scartato: l'utente vuole che l'agente segua la lingua in cui gli si scrive, non
quella impostata nel profilo. Non riproporlo. Il task DevOps 2325 chiede "italiano fisso
salvo richiesta esplicita": è una posizione da discutere con RINA, non un difetto da
correggere.

## 15/10: PT e FR falliscono, il cinese no — riprodotto in F&O

Segnalazione di Emanuela (PT risponde in inglese, FR in italiano, cinese "parziale").
Verificato in F&O su RSSE, **chat appena ricaricata**, quindi non e' trascinamento di contesto:

- **portoghese** ("preciso de um codigo para consultoria informatica") -> risposta in **italiano**
  ("Alternative piu' specifiche disponibili")
- **francese** ("j'ai besoin d'un code pour le nettoyage des bureaux") -> risposta in **italiano**
- **cinese** -> risposta **corretta in cinese** ("其他实验室相关代码：")

Le regole ci sono e sono generiche in **entrambi** gli agenti: l'orchestratore ha un blocco
LANGUAGE esplicito ("Always respond in the language of the user's current message... Do not use
system locale, environment language, previous turns"), il figlio pure. Non e' un problema di
regola mancante ne' di delega.

**Pattern:** falliscono le lingue romanze vicine all'italiano, non quelle lontane. Il modello
scivola sull'italiano quando la distanza linguistica e' piccola.

**Il "parziale" del cinese e' un'altra cosa:** la risposta e' in cinese ma le **descrizioni degli
articoli restano in italiano**, perche' vengono dal dato F&O. A chi legge cinese sembra una
risposta a meta'. Collegato a [[rina-tassonomia-nuova-senza-descrizioni]]: il campo descrizione
per lingua non e' popolato.

## Tentativo fallito del 15/10: riscrivere la sezione LANGUAGE non basta

Le istruzioni dell'orchestratore erano a **7990/8000**: nessuno spazio per aggiungere. Ho quindi
**riscritto** la sezione LANGUAGE a parita' di caratteri, rendendola operativa invece che
dichiarativa ("First identify the language of the current user message, then write the whole
answer in it... Never answer in Italian or English unless the user wrote in that language").
Salvato (7989) e pubblicato.

**Non ha funzionato.** Riprovati PT e FR in chat pulita: entrambi rispondono ancora in italiano.

Il dato diagnostico piu' utile e' la risposta francese: *"Per il **nettoyage des bureaux**
(pulizia uffici) nella societa' RSSE..."*. Il modello **riconosce** la lingua della domanda, la
cita, e sceglie comunque l'italiano. Non e' un problema di comprensione ne' di istruzione
mancante: e' una preferenza del modello che le istruzioni non piegano.

**Non insistere su questa leva.** Rollback in `ROLLBACK_orchestratore_sezione_LANGUAGE.md`.

## Secondo tentativo fallito: nemmeno il Response formatting piega la lingua

Il campo Response formatting dichiara di prevalere sulle istruzioni ("If these conflict with
other instructions for this agent, these will override") ed era la leva che aveva funzionato per
il prefisso `MMD |` e per l'elenco uno per riga. Provato il 15/10 mettendo
`Scrivi tutta la risposta nella lingua del messaggio dell'utente.` **in prima posizione**,
comprimendo le regole di formato per far spazio (479/500). Pubblicato.

**Non ha funzionato**: il portoghese risponde ancora in italiano. **E ha peggiorato il formato**:
comprimendo la regola delle due sezioni, gli articoli sono usciti tutti su una riga sola invece
che uno per riga. Ripristinato il testo a 488/500 e ripubblicato.

**Conclusione dopo due leve diverse:** la lingua di risposta sulle lingue vicine all'italiano non
e' governabile ne' dalle istruzioni ne' dal Response formatting. Da portare a RINA e,
eventualmente, a Microsoft insieme al ticket gia' aperto. Non spendere altro tempo sui prompt.

## RISOLTO il 15/10: il Response formatting va scritto in INGLESE

**Causa:** il campo Response formatting dell'orchestratore era scritto **in italiano**. Quel campo
prevale sulle istruzioni ("these will override") e di fatto funzionava da istruzione di lingua
implicita: il modello componeva la risposta nella lingua in cui erano scritte le regole di forma.

**Fix:** riscritto lo stesso contenuto **in inglese** ("Write the whole answer in the language of
the user's message. Start every answer with "MMD | "... "). Pubblicato.

**Prova:** domanda in portoghese in F&O -> risposta **in portoghese**
("Itens disponiveis na sua empresa... Itens presentes no Dynamics mas a solicitar").

**Regola generale da tenere:** i campi di configurazione che governano forma e stile vanno scritti
in **inglese**, non nella lingua di uno degli utenti, altrimenti inchiodano la lingua di risposta.
Vale per Response formatting e, presumibilmente, per le istruzioni.

