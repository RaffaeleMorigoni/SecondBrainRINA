---
name: rina-capex-opex-knowledge
description: "Dall'08/10/2026 la guida CAPEX/OPEX e' nella knowledge dell'agente RINA; copre la classificazione per categoria ma non e' una policy AFC ufficiale"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-08T14:22:16.347Z
---

`Summary capex e opex 1.pdf` e' caricato nella knowledge dell'agente (stato **Ready**,
pubblicato l'08/10/2026). Fonte: SharePoint `SupplyChainPlatform-AgenteAIMMDPrivato`, allegato
da Emanuela alla riga 6 del registro UAT dell'8 ottobre.

**Contenuto:** tabella CAPEX/OPEX su dieci categorie (software, progetti in corso, migliorie su
beni di terzi, fabbricati, manutenzioni, impianti, attrezzature, mobili, macchine elettroniche,
automezzi) piu' la regola pratica: crea un nuovo bene o ne aumenta valore/vita utile → CAPEX;
gestione ordinaria, manutenzione o canone ricorrente → OPEX.

**Il disclaimer conta.** La seconda pagina dice che e' "una sintesi semplificata", **non una
policy AFC ufficiale**, e che "non deve essere divulgato o assunto come riferimento normativo o
contabile". La sua finalita' dichiarata e' pero' proprio il supporto a chi apre RDA, cioe' quello
che fa l'agente. Per questo in `F&O OPERATIONAL DATA` e' stata aggiunta la riga:
`CAPEX/OPEX advice is an operational summary, not AFC policy: say so and send doubtful cases to Finance.`

**Cosa copre e cosa no.** Copre la classificazione per categoria: il requisito BRD della Fase 6
passa da KO a **Parziale**. **Non** contiene soglie di importo ne' vite utili: il caso di test che
le chiede deve continuare ad attendersi che l'agente dichiari di non averle.

**Verificato in UAT l'08/10:**
- "devo noleggiare un'auto aziendale per un dipendente..." → SE401001.0001, tipo `1 - I - Costo
  OPEX indiretto`, conclude OPEX e **chiude con il rimando a Finance**;
- "licenza software perpetua invece che abbonamento annuale" → ME302001.0001 - Licenze Perpetue
  Software, due tipi contabili, suggerisce `3 - I - Investimento CAPEX` e **spiega che il canone
  e' OPEX mentre la perpetua e' CAPEX**: e' il criterio che sta solo nel PDF, quindi la knowledge
  viene davvero consultata. In questa risposta pero' **il rimando a Finance non compare**: la riga
  del disclaimer e' intermittente.
- La domanda concettuale secca ("il noleggio di un'auto aziendale e' capex o opex?") cade invece
  nel fallback generico: vedi [[rina-fallback-generico-orchestratore]]. **La knowledge serve solo
  se la domanda arriva all'agente**, e ci arriva quando e' ancorata a un acquisto o a un articolo.

## 21/10/2026 - ora il PDF e' su DUE agenti, e l'equivoco si spiega cosi'

Verificato in Copilot Studio:

| agente | file | caricato da | quando |
|---|---|---|---|
| **child** `RINA Procurement & MMD Assistant` | `Summary capex e opex 1.pdf` | **Raffaele** | 12 giorni fa (~9/10) |
| **padre** `Copilot for finance and operations apps` | `Summary capex e opex 1 (1).pdf` | **Chiara** | 22 ore fa (20/10) |

Entrambi in stato Ready. **Non era mai sparito**: era sul child dall'inizio. Chiara ha cercato
nel **padre**, dove non c'era, ha concluso che mancasse e ce l'ha caricato lei. In chat ha
scritto di averlo messo "nel child", ma la knowledge del child e' ferma a Raffaele/12 giorni fa:
l'ha messo sul **padre**. Il child risulta modificato il 15/10, il padre il 20/10.

**Non e' detto sia un errore**: la knowledge sul padre potrebbe risolvere il caso della domanda
concettuale secca che cadeva nel fallback (vedi sopra e [[rina-fallback-generico-orchestratore]]),
perche' a quella risponde il padre. Ma ora il documento e' citabile da due posti: se ricompaiono
citazioni doppie o il marcatore «1» isolato, guardare prima qui.

**ATTENZIONE - istruzioni del padre a 7.988 / 8.000 caratteri.** Venerdi' erano 7.968. Restano
**12 caratteri**: la prossima aggiunta tronca in silenzio l'ultimo blocco, che e'
`ONE ANSWER PER TURN` (vedi [[copilot-studio-istruzioni-limite-8000]]). Verificato il 21/10 che
`LANGUAGE` e `PROCUREMENT RESULTS` sono **intatti**, con la regola
"Write it in the user's language" ancora al suo posto: il lavoro sulle lingue del 16-19/10 non e'
stato toccato.

## 21/10/2026 pomeriggio - il task 2422 verificato, e il 2424 lavorato

**Il 2422 funziona e chiude il buco documentato sopra.** La domanda concettuale secca
("il noleggio di un'auto aziendale e' capex o opex?") **non cade piu' nel fallback**: caricare il
PDF anche sul **padre** era la mossa giusta, perche' a quel tipo di domanda risponde lui. Vedi
[[rina-fallback-generico-orchestratore]], che su questo punto va considerato superato.

**Attenzione, fatto nuovo:** il blocco `F&O OPERATIONAL DATA` con la riga
`CAPEX/OPEX advice is an operational summary, not AFC policy: say so and send doubtful cases to Finance`
**non esiste piu'** nel prompt dell'agente connesso. Rimosso in una riscrittura fra l'08/10 e il
20/10. Prima delle correzioni del 21/10 la parola "Finance" compariva **una sola volta in 29.009
caratteri**, e solo nel titolo "Dynamics 365 Finance and Operations". E' questo che aveva spento
il rimando a Finance, non l'intermittenza del modello.

**Corretto il 21/10 (task 2424)**, con cinque sostituzioni mirate sul prompt dell'agente connesso:
via ogni "the official RINA AFC guidelines" -> "the CAPEX/OPEX classification summary", piu' la
qualificazione "simplified operational summary, not official AFC policy" e il rimando a Finance.
Zero occorrenze residue di `official RINA AFC`, `AFC guidelines`, `AFC criteria`. Campo da 29.009
a 29.327 caratteri. Backup: `plans/backup/20261021_prompt_child_ORIGINALE.txt`.

**Il prompt contiene DUE blocchi sullo stesso tema**: `OFFICIAL DOCUMENTATION` e
`AFC CAPEX/OPEX guidelines`, mille caratteri piu' avanti. Non accorpati (non sono nostri), ma e'
il primo posto da guardare se le risposte CAPEX/OPEX si comportano in modo incoerente.

**Cosa NON si e' risolto.** Sulla domanda concettuale secca la risposta chiude ancora con
"la classificazione finale va verificata secondo le linee guida AFC applicabili", senza la frase
sulla sintesi operativa e senza Finance. Una riga aggiunta anche sull'**orchestratore**
(`CAPEX/OPEX guidance from Knowledge is a simplified operational summary, not AFC policy…`)
**non ha cambiato quella risposta**. Ripetere la regola nelle istruzioni non basta: se serve che
il rimando a Finance sia costante, va intercettato altrove.

**Rettifica in giornata (21/10, ore 15).** Avevo concluso che il rimando a Finance non fosse
rientrato: **sbagliato**, vale solo per la domanda sul noleggio auto. Alla domanda
"come contabilizzo un servizio di manutenzione software annuale?" la risposta chiude con
"la valutazione finale resta di competenza Finance", e quella sulla consulenza informatica dice
"in base alla regola operativa RINA" al posto di "le linee guida AFC". **Le correzioni al prompt
funzionano**; quello che resta e' che su alcune formulazioni il disclaimer non compare.

## 29/10/2026 - verificato in F&O da Emanuela, comportamento pieno su 4 turni

Conversazione reale (schermate in `plans/prove_uat/`): partenza da `SE108001.0001`, poi
"non mi ricordo la differenza", poi "c'e' una linea guida di finance?", poi "posso visualizzare
questa guida?". **Tutti e quattro i turni corretti**: label MMD, italiano, contesto del primo
turno mantenuto, disclaimer "non e' una policy AFC ufficiale" e rimando a Finance presenti.
Il disclaimer intermittente di cui sopra, qui, non si e' visto.

**"Hive" non e' un'allucinazione**: sta nel prompt del figlio —
*"Refer to official RINA documentation on Hive when the required information is not available in
the agent's Knowledge."* Se un tester lo segnala come inventato, e' voluto.

**Unica frase imprecisa**, alla richiesta di vedere il documento: *"Non dispongo del contenuto
integrale o di un collegamento diretto da mostrare."* Il contenuto **ce l'ha** (e' nella
knowledge); quello che non ha e' il file da aprire. Dettaglio di fraseggio, non un difetto.

**La knowledge non e' un allegato**: l'agente non cita la fonte, non mostra il PDF e non lo
linka. Chi ha fornito il documento si aspetta di rivederlo e non succedera' mai: va spiegato,
non corretto.
