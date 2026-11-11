---
name: rina-padre-copia-non-traduce
description: la prosa restava italiana perche' le istruzioni del padre gli vietavano di tradurre il resolvedResult del figlio
metadata:
  type: project
---

Trovato il 16/10/2026, ultima sezione delle istruzioni dell'orchestratore (ONE ANSWER PER TURN),
blocco PROCUREMENT RESULTS:

> "that agent never writes to the user. Your reply for the turn is its resolvedResult, sent once
> and **copied as it is** ... **Never translate or reformat it** and never drop its MMD label"

Il padre **non compone** la risposta di ricerca: ricopia quella del figlio, con divieto esplicito
di tradurre. Per questo tre riscritture del Response formatting non avevano spostato la prosa,
mentre le domande di knowledge (CAPEX/OPEX), a cui risponde il padre, erano sempre nella lingua
giusta.

**Correzione pubblicata** (7998 -> 7997 caratteri, rollback in
`ROLLBACK_istruzioni_padre_traduzione.md`):
`Never translate or reformat it and never drop its MMD label`
->
`Write it in the user's language, keep layout and MMD label`

**Esito**: francese -> risposta interamente in francese, elenco puntato intatto, etichetta MMD |
al suo posto. Italiano -> nessuna regressione, elenco uno per riga.
Prove: `plans/prove_uat/20261016_FR_microscopi_TUTTO_FRANCESE.png` e
`20261016_IT_regressione_dopo_fix_prosa_ok.png`.

**Da sorvegliare**: e' sparita la parola "reformat", quindi il padre potrebbe riformattare
l'elenco. Se riemerge il paragrafo unico (bug 2334), rimettere il testo precedente.

Insieme a [[rina-traduzioni-prodotto-esistono]] chiude [[rina-lingua-elenco-vs-prosa]].

## Giro completo sulle lingue (16/10/2026 pomeriggio)

Stessa domanda (microscopi elettronici), chat pulita, societa' RSSE:

| lingua | prosa | descrizioni |
|---|---|---|
| italiano | ok | ok |
| francese | ok | ok |
| portoghese | ok | ok |
| cinese | ok | ok |
| inglese | **italiano** | inglese |
| spagnolo | **italiano** | **italiano** |

Due cose emerse durante il giro:
1. L'input UserLanguage arrivava a volte 'it' anche con domanda in portoghese: l'AI dichiarava la
   lingua delle keyword tradotte invece di quella del messaggio. Corretto con una riga nelle
   istruzioni del padre, ricavando spazio dalla frase "Tool data is stored in Italian..." ormai
   obsoleta: `Pass the user's own message language to the search tool, not that of the
   translated keywords.` (7997 -> 7975).
2. Togliendo "reformat" era riemersa la riga grezza (`Descrizione: ... Tipo: Goods Unita: QTY`).
   Ripristinato il vincolo mantenendo la traduzione:
   `Write it in the user's language, never reformat it or drop its MMD label` (7975 -> 7989).

**Lo spagnolo e' debole nei dati**: in anagrafica diversi codici hanno la traduzione `es` vuota o
scritta in inglese. Non e' un difetto dell'agente.

**Corretto il 16/10**: in `Seleziona articoli` il ripiego usava `coalesce`, che NON scarta la
stringa vuota: con una traduzione esistente ma vuota (caso spagnolo) la descrizione sarebbe
uscita vuota. Ora:
`replace(if(empty(coalesce(outputs('Mappa_traduzioni')?[item()?['ItemNumber']], '')), coalesce(item()?['SearchName'], ''), outputs('Mappa_traduzioni')?[item()?['ItemNumber']]), '"', '''')`
Pubblicato e verificato in italiano: nessuna regressione.

## Regressione e ripristino (16/10, fine pomeriggio)

La riga `Pass the user's own message language to the search tool...` aggiunta nelle istruzioni del
padre ha provocato una **regressione grave**: l'agente chiedeva all'utente *"Qual e' la lingua in
cui stai scrivendo il messaggio?"* invece di dedurla. Riprodotto 2 volte in portoghese.
Aggiungere `never ask for it` NON e' bastato: l'input e' riempito dall'AI e, quando e' incerta,
Copilot Studio chiede all'utente. E' il comportamento standard degli input "Dynamically fill with AI".

**Riga rimossa** (7995 -> 7895). Dopo il ripristino il portoghese funziona: prosa e descrizioni in
portoghese, elenco pulito. Quindi quella riga non serviva: il miglioramento sul portoghese era
venuto dalla descrizione dell'input nel flusso, non da lei.

**Lezione**: non aggiungere istruzioni che spingono il modello a "procurarsi" il valore di un
input riempito dall'AI; lo porta a chiederlo all'utente.

## Riga grezza sul risultato singolo (16/10, 17:31)

Con UN SOLO risultato l'agente riversava i campi del dato strutturato come etichette:
`Codice: SE707005.0001 Descrizione: ... Tipo: Servizio Unita: LUMPSUM`. Con l'elenco multiplo no.
`never reformat it` non bastava a coprirlo.

Aggiunta nelle istruzioni del padre, dopo la regola sul formato (7895 -> 7968):
`Present each item as code plus description, never as field: value pairs.`

Verificato: risultato singolo -> `MMD | SE707005.0001 Application development and configuration`,
niente etichette; elenco multiplo in italiano -> intatto, uno per riga.

## Inglese: tentativo fallito (16/10, 17:45)

Aggiunto in LANGUAGE, dopo "it is authoritative": ` Any language, this one included.`
(formulazione generica, per dire che la regola vale anche per la lingua in cui sono scritte le
istruzioni). Spazio ricavato accorciando la regola sul formato in
`Each item: code plus description, never field: value pairs.` Totale 7988.

**Non ha funzionato**: domanda in inglese -> descrizione corretta in inglese, prosa ancora in
italiano. Nessuna regressione (francese verificato ok, formato senza campi tecnici ok).

Riepilogo dei tentativi falliti sull'inglese: tre riscritture del Response formatting, la regola
generica qui sopra. Ha funzionato solo la rimozione di `never translate it`, che ha risolto
francese, portoghese e cinese ma non l'inglese.
**Da indagare con metodo diverso, non con altri tentativi sul prompt.**

## Analisi sull'inglese (16/10, 18:00) - causa isolata, non risolta

**Fatto accertato con test discriminante**: in inglese, domanda di knowledge (CAPEX/OPEX) ->
risposta in inglese perfetto; domanda che innesca la ricerca -> prosa italiana. Quindi l'inglese
non e' trattato diversamente dal modello: e' il percorso con la ricerca a rompersi, come
accadeva a FR/PT prima della rimozione di `never translate it`.

**Ipotesi** (NON confermata): dopo l'introduzione delle traduzioni da anagrafica, per l'inglese
il testo che arriva al padre e' misto - descrizioni gia' in inglese + prosa italiana - e il padre,
vedendo la parte sostanziale gia' nella lingua giusta, non ritraduce il contorno. Per FR/PT/ZH
lo scarto e' totale e la traduzione scatta.

**Tentativo fatto e ANNULLATO**: riscrittura nelle istruzioni dell'agente connesso
(agents/b4a1f29d-..., 27.974 caratteri - NON la pagina standalone del figlio, che e' un'altra
cosa) della frase "Item descriptions ... come from Dynamics in their original language, usually
English ...", piu' una frase sul fatto che le frasi di contorno seguono sempre la lingua utente.
**Ha peggiorato il francese** (prosa tornata italiana) senza sistemare l'inglese.
Ripristinato il testo esatto, 27.974 caratteri, verificato: francese di nuovo corretto.

**Tentativi falliti sull'inglese, in totale cinque**: tre riscritture del Response formatting,
la regola generica `Any language, this one included.`, la riscrittura nel figlio.
Non toccare piu' a tentoni: serve vedere il resolvedResult reale in una sessione inglese.

## Scoperta decisiva: pannello di test vs F&O (16/10, 18:30)

Nel **pannello di test** di Copilot Studio la stessa domanda inglese riceve risposta
**in inglese**; in **F&O** no. La traccia del test mostra che il padre passa correttamente
`UserLanguage = en` al figlio.

Quindi: istruzioni, flusso e passaggio della lingua sono corretti. La differenza e' il
**contesto di sessione F&O**, la cui interfaccia e' in italiano. Per francese, portoghese e
cinese il segnale della lingua utente e' abbastanza forte da prevalere; per l'inglese no.

**Sesto tentativo** (pubblicato e lasciato): nelle istruzioni dell'agente connesso,
`Ignore UserLanguage, the application interface language and any system setting whenever they
differ from the language the user actually wrote in.`
->
`When UserLanguage is provided, trust it: the parent computed it from the user's own words.
Ignore the application interface language and any system setting whenever they differ from it.`
Concettualmente piu' sensato (prima il figlio ignorava un valore corretto), ma **non risolve
l'inglese in F&O**. Nessuna regressione: francese verificato ok dopo la modifica.

**Conclusione**: non e' un problema di prompt. Il fattore e' il contesto F&O. Possibili strade
vere: verificare la lingua di interfaccia dell'utenza F&O usata per i test (se fosse inglese il
caso sparirebbe), oppure portare il punto a Microsoft come limite della piattaforma.

## 29/10/2026 — settimo tentativo fallito, e il difetto e' RIAPERTO

**Il 19/10 l'inglese era dato per risolto ([[rina-lingua-interfaccia-fo]]): oggi non lo e'.**
Misure in F&O, RSSE, interfaccia italiana, sessione nuova ogni volta:

| domanda | descrizioni | prosa |
|---|---|---|
| fr — "Quel code MMD... developpement de logiciels ?" | ok | **francese** |
| en — "Which MMD code should I use for software development?" | inglese | **italiana** |
| en — "...maintenance work on mechanical systems at height" | inglese | **italiana** |

Il francese regge: **non e' una regressione generale**, cede solo la prosa e solo in inglese.
Il codice e la descrizione sono corretti, quindi non e' la ricerca.

**Settimo tentativo, fallito e ripristinato.** Ipotesi: la frase del figlio
*"ignore the language of the Task the orchestrator writes: **the Task is often in English while
the user is not**"* insegna al modello che l'inglese che vede non e' la lingua dell'utente — e con
un utente che scrive davvero in inglese gli si ritorce contro. Spiegava l'asimmetria, ma
sostituirla con una formulazione simmetrica **non ha cambiato nulla**. Testo riportato
all'originale (29.447 caratteri) e ripubblicato: nessuna modifica lasciata in piedi.

**Elenco dei tentativi falliti, per non ripeterli**: tre riscritture del Response formatting,
`Any language, this one included.` nel padre, la riscrittura di "usually English" nel figlio
(che ruppe il francese), il trust di UserLanguage, e ora la simmetria sul Task.

**Il passo che manca non e' un altro tentativo**: leggere il **`resolvedResult` reale** di una
sessione inglese in `Monitor > Sessions`. Dice se la prosa italiana nasce nel figlio o se il
figlio scrive inglese e il padre non ricopia: due difetti diversi, due soluzioni diverse.
Attenzione: **Sessions si aggiorna ogni 15 minuti**, e nel pannello di test il difetto **non si
riproduce**.

## 29/10 — LOCALIZZATO: la prosa italiana nasce nel FIGLIO, non nel padre

Quattro fatti, misurati oggi:

1. **Il padre l'inglese lo sa**: domanda inglese che non passa dal figlio (*"What is the
   difference between capex and opex?"*, dalla Knowledge) → risposta **interamente in inglese**.
2. **Nessuno traduce verso l'italiano**: l'ordine del padre e' *"Write it in the user's
   language"*. Se ricevesse inglese non produrrebbe italiano. L'italiano e' quindi la lingua di
   **prima scrittura**, non una traduzione.
3. **Le etichette italiane non stanno in nessun prompt**: `Tipo`, `Categoria`, `Unita' di
   misura`, `Prezzo`, `Societa'` → zero occorrenze nelle istruzioni del figlio. Le **genera** il
   modello, in italiano. Quella scheda la compone il figlio.
4. **Il figlio sa cambiare lingua**: in francese la prosa esce francese.

In piu': la **Primary language** dell'agente e' `English (United States)`, l'italiano e' solo una
secondaria. Nemmeno la configurazione spinge verso l'italiano.

**Ipotesi residua sul perche' proprio l'inglese**: il figlio ha l'ordine di ignorare la lingua
del Task, che e' quasi sempre inglese; quando l'utente scrive in inglese quel segnale coincide
con quello da ignorare, e resta solo l'italiano dei dati. Toglierla **non e' bastato**.

**`Monitor > Sessions` mostra solo il dialogo**, non il `resolvedResult` ne' le chiamate agli
strumenti: non serve per questa diagnosi. Per la prova diretta servono Application Insights o un
marcatore temporaneo nella risposta del figlio.

## 29/10 — e' il PADRE a comporre la risposta; ottavo tentativo fallito

**Correzione a quanto scritto sopra lo stesso giorno.** La traccia del pannello di test mostra
`Execution summary` dello strumento `RINA - Search Items` con i suoi output, e **nessun
`resolvedResult`**: per le ricerche il padre chiama **direttamente lo strumento**, il figlio non
entra. Un marcatore diagnostico messo nel figlio **non e' mai comparso**, ne' in F&O ne' nel test:
conferma. Buona parte degli otto tentativi ha quindi modificato **l'agente sbagliato**.

**Ottavo tentativo, sul padre, fallito.** Il blocco `PROCUREMENT RESULTS` aveva due ordini in
conflitto nella stessa frase: *"sent once and **copied as it is**"* e *"**Write it in the user's
language**"*, per giunta riferiti a un `resolvedResult` che in quel percorso non esiste. Tolto
"copied as it is" e aggiunta la regola per il caso reale (7.859 → 7.937): **nessun effetto**.
Ripristinato a 7.859 e ripubblicato.

**Conclusione operativa: non e' governabile dal prompt.** A parita' di istruzioni e modello, nel
pannello di test la stessa domanda esce in **inglese corretto**, in F&O no. La variabile e' il
**contesto di sessione F&O** — conferma [[rina-formato-fo-non-governabile]].

**Smettere di riscrivere regole sulla lingua: otto tentativi, zero spostamenti.** Portarlo come
limite della piattaforma, con le prove dei due canali a confronto.
Dettaglio: `plans/backup/20261029_ROLLBACK_padre_copied_as_it_is.md`.
