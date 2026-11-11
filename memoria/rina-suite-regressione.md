---
name: rina-suite-regressione
description: Suite di casi da rieseguire sull'agente RINA dopo ogni modifica alle istruzioni e dopo ogni aggiornamento di F&O, connettore o agente Microsoft
metadata:
  type: project
---

Rieseguire **sempre in sessione di test nuova**. Circa mezz'ora in tutto. Societa di riferimento
**RSSE** salvo dove indicato.

**A. Caso canonico (il piu sensibile alle regressioni)**
"Che codice devo usare per la manutenzione di attrezzature?" → **SE152002.0001**, mai SE105001.0001;
due sezioni distinte; **SE152001.0001** fra quelli da richiedere; nessuna classe merciologica della
vecchia nomenclatura; prefisso esatto `MMD | `; 40-45 secondi.

**B. Codici e contabilizzazioni**
| Richiesta | Atteso |
|---|---|
| acquisto di reagenti chimici | MA101021.0001 |
| prestazione professionale in ambito 9001 | PS101002.0002 |
| sviluppo di soluzioni informatiche a capex | SE302501.0001 **e** SE707005.0001 |
| servizio di consulenza informatica | SE707001.0001 piu le alternative SE7070xx |
| attrezzatura e materiali da laboratorio | ME201001.0001 e ME201003.0001 |
| intermediazione immobiliare | SE601001.0001 e SE601002.0001 |
| codice MA101008.0002, contabilizzazione e conti | conto 410000, contabilizzazione 1, project category Goods |
| acquisto che prima caricavo sul codice Npprom141 | MA101004.0001, senza mai citare il codice storico |

**C. Forma della risposta** (i difetti chiusi il 29/09, tutti verificabili solo guardando la forma)
1. **Etichetta**: ogni risposta inizia con `MMD | ` esatto. Verificare **anche al secondo turno** e
   **sulle risposte dei conti contabili**, dove si rompeva.
2. **Lingua**: domanda in italiano → risposta in italiano, e **mai** una domanda sulla preferenza
   linguistica. Verificare con due domande di fila nella stessa sessione.
3. **Messaggio unico**: una sola risposta per turno. Se ne compaiono due, quella senza etichetta e
   dell'orchestratore: e il punto aperto 28.
4. **Sezioni sempre presenti**: vedi punto D.

**D. Prova su societa senza articoli** (intercetta difetti invisibili su RSSE)
Cambiare temporaneamente la societa predefinita di `C_110912` in **RSPA** con un PATCH OData su
`SystemUsers` (`If-Match: *`, risponde 204), rieseguire **A** e **prestazione ISO 9001**, poi
**ripristinare RSSE**. Atteso: stessi codici, tutti nella sezione da richiedere, sezione dei
disponibili presente con la sua intestazione e una riga che dichiara l'assenza, etichetta corretta.
Se la risposta torna a essere un paragrafo discorsivo senza etichetta, e la regressione del punto 39.

**E. Casi noti che non devono ricomparire**
- nessun codice inventato: ogni codice mostrato deve esistere in Dynamics
- nessuna classe merciologica della vecchia nomenclatura (Np, Pr, CS, In) nella risposta
- nessuna legal entity nominata accanto ai codici da richiedere
- "laboratorio" da solo: l'agente chiede di precisare. **Oggi risponde in inglese e senza etichetta:
  e il punto aperto 35**, non ancora corretto

**05/10/2026 - metodi di valutazione.** Il test set usava solo *Answer quality*, che **non legge
l'Expected response**: boccia come `Not answered` ogni rifiuto, anche quando rifiutare e' la
risposta giusta, e non verifica mai i vincoli di forma (etichetta `MMD | `). Aggiunto
**Meaning match** (pass score 50/100), che confronta davvero con l'atteso. Da leggere Meaning match
per i casi "deve rifiutare" e Answer quality per la fondatezza.

**Ricerca inversa conto -> MMD:** non implementata. Il dato corretto per il conto 411311 e'
SE704002.0001 - Consulenze bilancio (riga unica nella matrice del file old_new_with_matrix.xlsx):
tenerlo fuori dall'atteso finche' la funzionalita' non esiste, altrimenti Meaning match pretende
che compaia nella risposta.

**Enumerazione da proprieta' (CAPEX): difetto intermittente.** Alla domanda "quali codici MMD
hanno tipo di contabilizzazione 3 Capex" a volte l'agente rifiuta correttamente, a volte elenca
una decina di codici che non ha interrogato (ME301002.0001, ME302001.0001, ...). *Answer quality*
premia l'elenco inventato con un Pass; **solo Meaning match lo boccia** (25/100). E' la ragione
per cui il secondo metodo serve: senza, il difetto passa inosservato.

**06/10/2026 - clausola contro l'enumerazione.** Aggiunta in coda a VERIFIED CODES ONLY nelle
istruzioni dell'agente figlio: funziona (CAPEX e conto 411311 passano Meaning match con 100).
Attenzione al confine: la prima stesura vietava di partire da "a given class" e ha fatto rifiutare
anche `codici della classe merceologica EX10`, che e' una richiesta legittima. La classe/categoria
merceologica **e' interrogabile** dal flusso di ricerca; non lo sono il tipo di contabilizzazione e
il conto contabile. La clausola ora lo dice esplicitamente.

**Leggere i due metodi separatamente.** Dopo la clausola, Answer quality scende (79%) perche' conta
ogni rifiuto come `Not answered`, mentre Meaning match sale (92%). Per i casi "deve rifiutare" vale
Meaning match; Answer quality resta utile per la fondatezza delle risposte vere.

**Versione definitiva della clausola (06/10/2026).** Due eccezioni esplicite, entrambe nate da
regressioni osservate: (1) categoria o classe merceologica non sono attributi contabili, la
richiesta che ne nomina una e' una ricerca normale; (2) una richiesta fatta di parole comuni, anche
una sola parola, e' una ricerca per parola chiave: va eseguita, e solo dopo aver elencato i
risultati si invita a precisare - mai rispondere chiedendo di precisare invece di cercare.
Senza la (2) il caso `laboratorio` smetteva di elencare. Esito: **Meaning match 100%**, Answer
quality 92% (i due soli Fail sono i rifiuti voluti), tutte e 24 le risposte con l'etichetta `MMD | `.

**06/10/2026 - la suite passa a 29 casi.** Aggiunti i cinque segnalati dai tester sul file di
feedback: varianti su SE201001.0001, articolo per partecipazione a un corso, prove di laboratorio
(i tre di Sara Ferrari, gia OK) e i due di Enrico sulla RDA per consulenza informatica. Inseriti
direttamente nel test set con **Add questions > Write**, senza reimportare il CSV: la via manuale
evita il limite dei 1000 caratteri e non duplica il set. Il CSV in Downloads e allineato a 29.

La quinta domanda e stata resa autonoma ("Per una consulenza informatica nello sviluppo dei report,
quale MMD devo selezionare?"): il test set e a risposta singola e il follow-up senza contesto non
avrebbe senso. **Il caso della RDA fallira finche il passo 5 non e implementato**: l'atteso chiede
che proponga comunque il materiale, ed e voluto - serve a misurare la distanza dal requisito.

**Pass score 50 di Meaning match e troppo permissivo.** Nel run del 06/10 a 29 casi il punteggio
complessivo era 100%, ma sei casi stavano al minimo (50) e tre di quei sei erano risposte
sbagliate o incomplete: la RDA senza alcun materiale proposto, e i due casi da codice vecchio che
rispondevano "non e possibile fornire un NEW MMD". Un caso con l'etichetta `MMD | ` mancante ha
preso 75. **Non fidarsi del punteggio aggregato: guardare sempre i casi a 50 e 25.** Da valutare
l'innalzamento della soglia a 75.

**I verdetti dei singoli casi sono rumorosi: non inseguirli.** Il 06/10, due run consecutivi
della stessa suite hanno dato **entrambi 72%** di Meaning match ma con **insiemi di fallimenti
diversi**: quattro casi hanno cambiato verdetto fra due configurazioni che differivano per una
clausola irrilevante per quei casi. Un punteggio di 50 su un caso non e un difetto, e un confine.
Prima di attribuire una regressione a una modifica, verificare che il caso fallisca in **due run
consecutivi**: solo cosi si distingue il segnale dal rumore. Esempi di segnale vero perche ripetuto:
l'invenzione delle varianti (0 due volte) e la RDA non conforme (25-50 su tre run).

**Why:** il 06/10 ho corretto tre volte inseguendo differenze che erano variabilita, e il punteggio
e sceso da 79% a 72% per poi non risalire con il rollback.

**Why:** il 29/09 tre tentativi di correzione hanno prodotto altrettante regressioni non visibili
nel punto in cui si stava lavorando. La suite e l'unico modo per accorgersene prima dei tester.

**How to apply:** rieseguirla dopo ogni modifica alle istruzioni e dopo ogni aggiornamento di F&O,
del connettore Fin & Ops, dell'agente orchestratore Microsoft o del modello. Vedi
[[rina-test-regressione-manutenzione]] e [[rina-utenze-tester-societa]].

**Automatizzata in Copilot Studio il 29/09.** La suite e caricata come test set in
**Evaluation** (sezione gia usata nel progetto, con test set precedenti di Chiara): 13 casi,
tipo dati *Single response*, metodo *Answer quality*, profilo utente `C_110912@rina.org`.
Il CSV di origine e in `Downloads\RINA_Suite_Regressione_MMD.csv`, due colonne
`Question` e `Expected response`. Il caricamento del file va fatto **a mano dall'utente**:
`file_upload` non puo leggere file fuori dalla working directory della sessione.

**Primo run: 85%, 11 pass su 13.** I due fail sono entrambi i casi Npprom141 e **non sono difetti
di logica**: `FlowActionInternalServerError` / `BadGateway` sul flusso Search Items, a 17 secondi di
distanza, durante l'esecuzione in serie. Stesso genere del punto aperto 4. Da rieseguire isolati
per confermare.

**Limiti da tenere presenti:** *Answer quality* e una valutazione **semantica**, quindi tollerante.
Nel caso dello sviluppo software ha dato Pass pur essendo uscito solo `SE302501.0001` e non anche
`SE707005.0001`. Non sostituisce il controllo a occhio sui vincoli di forma. Restano inoltre fuori
dalla valutazione a domanda singola: le **sequenze di due domande** nella stessa conversazione e il
comportamento nelle **societa senza articoli rilasciati**.

**07/10/2026 - il codice vecchio non arrivava alla knowledge.** I due casi `Npprom141` e
`Npinfo521` fallivano (0 e 25) e li avevo attribuiti al blocco DLP: sbagliato, le knowledge
erano Ready. La traccia nel pannello di test mostra la causa vera: l'agente passava il codice
vecchio a `RINA - Search Items` come `Keywords`, lo cercava nel catalogo F&O dove per
definizione non esiste, e **non consultava mai le knowledge file**. La sezione
`CRITICAL - LEGACY MMD` diceva solo "use authoritative sources internally", troppo generico.
Riscritta per dire esplicitamente che il codice in nomenclatura vecchia si risolve dalla
knowledge e **non va passato alla ricerca articoli come parola chiave**, e che la ricerca si
fa dopo, sul codice corrente. Esito: entrambi i casi passano, `Npprom141` da `MA101004.0001`.

**Esito del run: Answer quality 86% (nuovo massimo), Meaning match 66%, Pass 19 / Fail 4.**
I quattro fallimenti residui: `MA101008.0002` (0, dato contabile da verificare sulla run
history del flusso), `Polizza assicurativa medica per collaboratori terzi` (50), e i due che
partono da attributo contabile - `tipo di contabilizzazione 3 Capex` e `conto 411311` (50
entrambi), che cadono nel fallback generico "Mi dispiace, non so come posso aiutarti" **senza
il prefisso `MMD | `**: cadono prima di arrivare alle istruzioni, quindi nessuna clausola li
raggiunge.

**Attenzione al contatore della run.** Resta fermo per minuti (14/29 per otto minuti) mentre
in realta avanza: la lista dei risultati si popola prima. Non interromperla credendola bloccata.

**07/10/2026 - MA101008.0002: e' un buco del dato in UAT, non un difetto dell'agente.**
Verificato sulla run history di `RINA - Get Item Accounting Types`. La prima azione del flusso,
`Lista categoria`, interroga `ProductCategoryAssignments` con
`$filter = ProductNumber eq 'MA101008.0002'` e risponde **200 con `"value": []`**: l'articolo
**non ha alcuna categoria di approvvigionamento assegnata**. Senza categoria la matrice
categoria -> contabilizzazione non ha chiave di ingresso, quindi `AccountingTypeCount = 0` e
nessun conto. Il flusso completa correttamente (40,98s) e l'agente riferisce fedelmente.

**Controprova sullo stesso flusso:** per `SE101001.0002` la stessa azione restituisce la riga
con `ProductCategoryName: "SE101001"` e `ProductCategoryHierarchyName: "RINA"`. La categoria
coincide col prefisso del codice articolo: per `MA101008.0002` dovrebbe essere `MA101008`, e
l'assegnazione manca.

L'atteso del caso (conto 410000, contabilizzazione 1, project category Goods) descrive quindi
un dato che **in UAT non esiste**: finche' l'assegnazione non viene caricata, il caso fallira'
sempre e il fallimento non misura l'agente. Da segnalare a Chiara insieme a
[[rina-rspa-anagrafica-mmd-vuota]] e [[rina-tassonomia-nuova-senza-descrizioni]]: e' la stessa
famiglia di anagrafiche incomplete in UAT.

**07/10/2026 - risolto: la risalita alla categoria radice ora c'e'.** Il fallback che la nota
HUDi dava per fatto ("risalita dal codice articolo alla classe merceologica tramite la radice
prima del punto") non era mai stato implementato. Nel flusso `RINA - Get Item Accounting Types`
l'azione `Lista categoria` interrogava `ProductCategoryAssignments` con
`$filter = ProductNumber eq '<item>'`: se l'articolo non ha assegnazione la risposta e `value: []`,
il `coalesce` sul RecId cade a 0 e `Lista matrice` cerca `EcoResCategoryId eq 0`, che non esiste.

Filtro nuovo, pubblicato in UAT:
`@concat('ProductNumber eq ''', trim(triggerBody()?['text']), ''' or ProductCategoryName eq ''', first(split(trim(triggerBody()?['text']), '.')), '''')`

Cerca l'articolo **oppure** una riga qualsiasi della categoria che porta il nome della radice:
il campo che serve e `AVA_ProductCategoryRecId`, identico sulle due righe, quindi il risultato
non cambia per gli articoli che l'assegnazione ce l'hanno. `LegalEntity eq 'ALL'` sulla matrice
c'era gia': quella parte della nota del tester era gia' recepita.

**Esito su MA101008.0002:** categoria MA101008, tipo `1 - I - Costo OPEX indiretto`, conto
`410000 - Materiale di consumo uso ufficio (Expense)`, project category Goods, societa RSSE —
esattamente l'atteso documentato. Il flusso scende da 40,98s a **2,08s**, perche' la query
trova subito una riga invece di scandire a vuoto.

**Limite residuo:** il fallback funziona solo se almeno un altro articolo di quella categoria ha
l'assegnazione. Se la categoria non compare in nessuna riga di `ProductCategoryAssignments`, si
torna al comportamento precedente.

**07/10/2026 - la suite completa: 47 casi.** Creato un **secondo test set**, distinto da quello
storico da 29 (che resta): 30 casi presi dal foglio `Wave 1 - Q&A` del file di feedback dei
tester (deduplicati: Roberto aveva ripetuto gli stessi quattro casi in tre tornate di ritest),
16 costruiti sui requisiti del BRD che nessun tester aveva toccato (vincoli 1.1, fasi 2-6,
Legal Entity, codice storico in frase lunga, capitolo 2 a/b/c, capitolo 3, riepilogo dello
schema), piu' `MA101008.0002` rimesso dopo la correzione del flusso. Metodi: **Answer quality +
Meaning match con pass score 70**. CSV in `Downloads\RINA_Suite_Tester_BRD.csv`.

**Caricare il CSV:** `file_upload` accetta solo file nella **working directory della sessione**
(Downloads e lo scratchpad vengono rifiutati). Copiarcelo, caricare, poi rimuoverlo.

**Attenzione a scrivere nella griglia dei casi.** Le celle diventano `textarea` solo quando sono
selezionate, e un click a occhio finisce quasi sempre sulla riga sbagliata: il 07/10 ho scritto
l'atteso di un caso dentro quello di un altro. Individuare la riga dal DOM
(`[role="row"]`, la nuova e' l'indice 1, subito sotto l'intestazione), ricavare il centro delle
sue due `[role="gridcell"]` e cliccare quelle coordinate, verificando `document.activeElement`
prima di digitare.

**Primo esito, 46 casi, Answer quality 76%, Pass 35 / Fail 11:** vedi
[[rina-fallback-generico-orchestratore]], perche' 6 degli 11 fallimenti sono un difetto solo.

**08/10/2026 - suite a 50 casi: Answer quality 80%, Meaning match 64%, Pass 28 / Fail 6.**
Miglior Meaning match mai misurato (era 57 la mattina, 55 il 7/10). Set aggiornato con: i due
attesi che erano scritti male (l'elenco ODA di `ME301002.0001` e il fornitore principale, spostato
da `MA101021.0001`, che ha zero acquisti, a `Npassi111`, che ne ha venti) e tre casi nuovi presi
dalle note di Emanuela sul registro UAT: inglesismi in frase italiana, elenco troncato da
dichiarare, licenza perpetua vs abbonamento per la guida CAPEX. **Tutti e tre passati al primo
colpo**, come i due attesi corretti.

**Dei 6 fallimenti, nessuno e' un difetto di contenuto:** quattro sono il fallback dell'orchestratore
(vedi [[rina-fallback-generico-orchestratore]]), uno e' un confine a 50, e uno e' il caso canonico
della manutenzione attrezzature finito a 0 per **`FlowActionTimedOut`**.

**Il timeout sotto carico e' un effetto della suite, non dell'agente.** Rieseguito subito dopo in
isolamento, lo stesso caso completa in **23,64 secondi** con la risposta corretta. Con 50 casi in
sequenza i tempi si allungano quanto basta a far scattare la soglia dell'agente su una domanda che
normalmente sta larga. **Prima di aprire un difetto per un caso andato in timeout nella suite,
rieseguirlo da solo.**

**08/10/2026 sera - run dopo lo spostamento del formato: Answer quality 82% (nuovo massimo),
Meaning match 52%, 50 casi in 28 minuti.** La suite ha fatto esattamente il suo mestiere: ha
trovato una regressione che in F&O non si vedeva.

**Il crollo di Meaning match non e' una regressione di contenuto.** I 27 fallimenti stanno **tutti
esattamente a 50**, compresi casi che poche ore prima erano a 100. Avendo tolto il tetto ai
risultati ("List every relevant result the tools returned"), l'agente elenca dieci codici dove
l'atteso ne nomina due: la sovrapposizione e' parziale e il giudice da' 50. **Quando una modifica
allunga sistematicamente le risposte, Meaning match scende senza che nulla sia peggiorato: leggere
la distribuzione dei punteggi, non la media.** Un blocco compatto tutto sullo stesso valore e'
la firma di un artefatto di misura.

**La regressione vera: la sezione "da richiedere" e' sparita.** Compare in **1 risposta su 50**.
Avevo rimosso il blocco `ANSWER FORMAT - MANDATORY` dalle istruzioni spostando le regole di forma
nel campo `Response formatting`, dopo averlo verificato **due volte in F&O**, dove funzionava.
Nel canale della valutazione no. **La conferma in un canale non vale per gli altri**: il campo
Response formatting non si sostituisce alle istruzioni ovunque.

**Due effetti collaterali dello stesso taglio**, entrambi su regole che il blocco rimosso teneva:
- sul caso canonico della manutenzione attrezzature ricompare **SE105001.0001**, che la suite
  dichiara non debba mai uscire;
- tre risposte espongono codici in nomenclatura vecchia - `Npcons730`, `Npinfo510`, `Npinfo620`
  su "software development", `Npcons410` su ISO 9001 - vietati dalle istruzioni ed e' la riga 31
  del registro UAT. (`Npassi111` nel caso del fornitore e' invece l'atteso, non un difetto.)

Prefisso `MMD | ` presente su **43 risposte su 50**.

**Da fare:** rimettere nelle istruzioni, in forma compatta, la regola delle due sezioni (articoli
rilasciati nella societa dell'utente / articoli in Dynamics da richiedere, entrambe presenti anche
se vuote), tenendo anche il campo Response formatting, e rieseguire. Punto di ripristino in
`scratchpad/ROLLBACK_istruzioni_answer_format.md`. Vedi [[rina-formato-fo-non-governabile]].

**09/10/2026 - run di conferma dopo il ripristino: Answer quality 78%, Meaning match 60%, 19 minuti.**
Rimessa nelle istruzioni, in forma compatta, la regola delle due sezioni, tenendo **anche** il campo
`Response formatting`. Confronto con la run precedente:

| | prima del taglio | dopo il taglio | dopo il ripristino |
|---|---|---|---|
| Answer quality | 80% | 82% | 78% |
| Meaning match | 64% | 52% | 60% |
| sezione "da richiedere" | - | 1/50 | 7/50 |
| prefisso `MMD \|` | - | 43/50 | 46/50 |
| codici legacy esposti | - | 3 | 2 (di cui 1 e' l'atteso) |

**Caso canonico rientrato:** `SE152002.0001` per primo, `SE152001.0001` nella sezione dei
richiedibili, `SE105001.0001` **sparito**. Resta `Fail 50` di Meaning match solo perche' la risposta
elenca sei codici dove l'atteso ne nomina due.

**Le regole di forma vanno tenute in DUE posti, non in uno.** Con le sole istruzioni la forma non si
applica in F&O; con il solo `Response formatting` la struttura a due sezioni sparisce negli altri
canali. La duplicazione, che il 08/10 avevo accusato di causare instabilita', non e' il problema.

**7/50 non e' un difetto:** la maggior parte dei 50 casi non e' una ricerca articoli (contabilita',
casi BRD, rifiuti voluti), quindi le due sezioni non c'entrano.

Vedi [[rina-formato-fo-non-governabile]].

**09/10/2026 - terza run, dopo la regola sulle categorie legacy: Answer quality 80%, Meaning match
52%, 39 minuti.** Serie completa delle tre esecuzioni della stessa suite in dodici ore:

| | taglio del blocco | ripristino | + regola categorie |
|---|---|---|---|
| Answer quality | 82% | 78% | 80% |
| Meaning match | 52% | 60% | 52% |
| sezione "da richiedere" | 1/50 | 7/50 | **11/50** |
| prefisso `MMD \|` | 43/50 | 46/50 | 43/50 |
| marcatore `[1]` | - | 10 | 9 |

**Cosa e' segnale e cosa e' rumore.** I due punteggi aggregati oscillano in una banda (78-82 e
52-60) senza seguire le modifiche: non vanno usati per decidere. L'unico indicatore che si muove
in modo monotono e' la **struttura a due sezioni, 1 -> 7 -> 11**, e conferma che la regola deve
stare nelle istruzioni. Il prefisso che scende da 46 a 43 senza che nulla lo riguardi e' la stessa
oscillazione gia' documentata: non inseguirla.

**La regola sulle categorie legacy non ha chiuso il difetto.** Aggiunto in `CRITICAL — LEGACY MMD`:
*"NEVER reveal a legacy identifier or a procurement category still named in the old nomenclature,
whatever tool returned it"*. Esito misto: `Npeste600`/`Npeste610` spariti dal caso della RDA di
intermediazione (che ora passa a 75 con entrambe le sezioni), ma sul caso ISO 9001 l'agente mostra
ancora `Npcons410` scrivendo *"come categoria di approvvigionamento"* - alla lettera cio' che la
regola vieta. Su tre run compare in due.

**How to apply:** quando un divieto scritto in istruzioni viene violato citando testualmente il
concetto che vieta, il problema non e' la formulazione: il dato arriva dallo strumento e il modello
lo considera legittimo. Smettere di riscrivere la regola e filtrare nel flusso. Vale anche per il
marcatore `[1]`: due difetti diversi, stessa lezione.
