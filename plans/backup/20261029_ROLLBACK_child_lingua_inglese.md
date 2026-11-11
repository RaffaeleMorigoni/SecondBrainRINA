# ROLLBACK — istruzioni del figlio, prosa inglese (29/10/2026)

Agente **RINA Procurement & MMD Assistant**, blocco `LANGUAGE AND LABEL ON EVERY TURN`.

## Frase ORIGINALE

```
Ignore UserLanguage completely and ignore the language of the Task the orchestrator writes: the Task is often in English while the user is not.
```

## Frase NUOVA

```
Ignore UserLanguage completely and ignore the language of the Task the orchestrator writes: the Task's language is often not the user's. The user's own language is authoritative also when it happens to be the same as the language of the Task or of these instructions.
```

## L'ipotesi

La frase originale insegna al modello un'associazione: **l'inglese che vede non e' la lingua
dell'utente**. E' scritta per il caso "Task in inglese, utente che scrive in altro". Quando
l'utente scrive **davvero** in inglese, la stessa regola gli dice di diffidarne, e lui ripiega
sulla lingua dominante del contesto, che e' l'italiano.

Spiega perche' falliva **solo** l'inglese: FR, PT, ZH non sono mai la lingua del Task, quindi la
regola non interferisce.

## Il quadro prima della modifica, misurato oggi

| domanda | descrizioni | prosa |
|---|---|---|
| fr — *"Quel code MMD... developpement de logiciels ?"* | ok | **francese, corretta** |
| en — *"Which MMD code should I use for software development?"* | inglese | **italiana** |
| en — *"...maintenance work on mechanical systems at height"* | inglese | **italiana** |

Nessuna regressione generale: la lingua funziona, cede solo sull'inglese.

**Nota storica**: sei tentativi precedenti hanno fallito su questo difetto (tre riscritture del
Response formatting, `Any language, this one included.` nel padre, la riscrittura di
"usually English" nel figlio, il trust di UserLanguage). **Nessuno aveva toccato questa frase.**

---

## ESITO: tentativo fallito, RIPRISTINATO alle 16:15

La modifica e' stata pubblicata, provata in F&O (RSSE, sessione nuova, stessa domanda inglese) e
**non ha cambiato nulla**: prosa ancora italiana, descrizione ancora inglese.
Istruzioni riportate a **29.447** caratteri, ripubblicate. Nessuna modifica lasciata in piedi.

**Settimo tentativo fallito** su questo difetto. L'ipotesi era buona e spiegava l'asimmetria, ma
non regge alla prova: va archiviata, non ripetuta.

## Quello che invece e' accertato oggi

- **non e' una regressione generale**: il francese esce in francese, prosa compresa;
- **non e' la ricerca**: il codice restituito e' quello giusto e la descrizione esce in inglese,
  quindi la catena delle traduzioni funziona;
- **cede solo la prosa, e solo in inglese.**

## Il passo che manca, e non e' un altro tentativo sul prompt

Leggere il **`resolvedResult` reale di una sessione inglese in F&O**: dice in un colpo solo se la
prosa italiana nasce nel figlio (e allora e' sua la regola da cambiare) o se il figlio scrive in
inglese e **il padre non ricopia**. Sono due difetti diversi con due soluzioni diverse, e
finora abbiamo sempre indovinato invece di guardare.

Si trova in **Monitor > Sessions**, che pero' si aggiorna **ogni 15 minuti**: la sessione delle
16:0x non era ancora disponibile al momento della prova. Da riprendere da li'.

Nel pannello di test il difetto **non si riproduce** (l'inglese esce in inglese), quindi quella
strada non serve.

---

## 29/10 ore 16:30 — DOVE NASCE LA PROSA ITALIANA: nel figlio

Non c'e' la traccia interna (vedi sotto), ma quattro fatti la localizzano senza ambiguita'.

**1. Il padre l'inglese lo sa, e lo usa.** Domanda inglese che **non passa dal figlio**
(*"What is the difference between capex and opex?"*, risposta dalla Knowledge): risposta
**interamente in inglese**, disclaimer e rimando a Finance compresi. Misurato oggi, non nel 2024.
Prova: `prove_uat/20261029_EN_knowledge_risposta_tutta_inglese.png`.

**2. Nessuno nella catena ha motivo di tradurre verso l'italiano.** L'istruzione del padre e'
`Write it in the user's language, never reformat it or drop its MMD label`. Se ricevesse un
resolvedResult inglese non produrrebbe italiano. L'italiano quindi non e' una traduzione: e' la
lingua in cui quel testo e' stato **scritto la prima volta**.

**3. Le etichette italiane non stanno in nessun prompt.** `Tipo`, `Categoria`, `Unita' di misura`,
`Prezzo`, `Societa'`: zero occorrenze nelle istruzioni del figlio (29.447 caratteri). Non sono
copiate da un template, le **genera il modello**, e le genera in italiano. Quella scheda la
compone il figlio.

**4. Il figlio sa cambiare lingua, e lo fa.** In francese la prosa esce francese. Quindi decide
lui la lingua della sua risposta, e sull'inglese decide male.

**In piu'**: la *Primary language* dell'agente e' **English (United States)**, l'italiano e' solo
una delle secondarie. Nemmeno la configurazione spinge verso l'italiano.

### Perche' proprio l'inglese

Il figlio ha l'ordine di **ignorare la lingua del Task**, che e' quasi sempre inglese. Quando
l'utente scrive in francese il suo messaggio e' un segnale forte e diverso; quando scrive in
inglese coincide con quello che gli e' stato detto di ignorare, e l'unico segnale forte che gli
resta e' **l'italiano dei dati** che riceve (SearchName, nomi di categoria, contesto F&O).
Resta un'ipotesi: toglierla dalle istruzioni **non e' bastato** (tentativo di oggi, ripristinato).

### Quello che Sessions NON da'

`Monitor > Sessions` mostra **solo il dialogo utente/agente**, non il `resolvedResult` del figlio
ne' le chiamate agli strumenti. La prova diretta richiede un'altra strada (Application Insights,
oppure far scrivere temporaneamente al figlio un marcatore della lingua scelta). Nel pannello di
test il difetto **non si riproduce**.
