# BASELINE — come si comporta l'agente PRIMA del cambio modello

**Rilevata il 21/10/2026, ore 17:00-17:30**, in F&O (non nel pannello di test).
**Modello: GPT-5.5 Chat** — in ritiro il 30/10/2026.
Tutte le prove sono **successive all'ultima pubblicazione della giornata**, quindi confrontabili.

Serve a una cosa sola: dopo il cambio modello, rifare questi sei casi e **confrontare**. Senza
questo "prima" scritto, dopo non sapremmo cosa e' cambiato.

---

## Caso 1 — Ricerca in italiano, RSSE

**Domanda:** *Devo acquistare un servizio di manutenzione per le attrezzature di laboratorio*

> MMD | Per un servizio di manutenzione delle attrezzature di laboratorio, gli articoli più
> pertinenti disponibili nella società RSSE sono:
> - SE153005.0001 Manutenzione altre apparecchiature specialistiche di laboratorio
> - SE153002.0001 Manutenzione impianti di laboratorio
> - SE153004.0001 Taratura strumentazione di laboratorio
> - SE153003.0001 Manutenzione microscopi elettronici (se applicabile alla specifica attrezzatura)
> - SE153001.0001 Manutenzione forni (se riferita a forni di laboratorio)
>
> In assenza di maggiori dettagli sul tipo di attrezzatura, SE153005.0001 … risulta generalmente
> la scelta più adatta.

**Cosa guardare dopo il cambio:** prefisso `MMD | `, elenco **uno per riga**, codice + descrizione
(mai `campo: valore`), lingua italiana, **cinque** codici, un solo messaggio.
Prova: `prove_uat/20261021_BASELINE_1_IT_manutenzione.png`

---

## Caso 2 — CAPEX/OPEX, domanda concettuale secca

**Domanda:** *Il noleggio di un'auto aziendale e' CAPEX o OPEX?*

> MMD | Il noleggio di un'auto aziendale è normalmente classificato come OPEX, perché rappresenta
> un costo operativo ricorrente per l'utilizzo di un bene che non entra nel patrimonio aziendale.
> Solo particolari strutture contrattuali potrebbero richiedere valutazioni differenti.
> **Questa indicazione si basa sulle linee guida operative CAPEX/OPEX disponibili e non costituisce
> una policy AFC ufficiale; nei casi dubbi è opportuno confermare la classificazione con Finance.**

**Cosa guardare dopo:** che la risposta **arrivi** (prima del 20/10 cadeva nel fallback), che dica
*"linee guida operative"* e **non** "le linee guida AFC", che **non** la presenti come policy
ufficiale e che **rimandi a Finance**. Sono le tre cose corrette sul task 2424.
Prova: `prove_uat/20261021_BASELINE_2_capex_concettuale.png`

---

## Caso 3 — Contabilizzazione servizi

**Domanda:** *Come viene contabilizzato un servizio di consulenza informatica?*

> …come OPEX, in quanto rappresenta un costo per servizi professionali utilizzati nell'operatività
> aziendale. Fanno eccezione i casi in cui la consulenza sia direttamente attribuibile a un
> progetto pluriennale di sviluppo che genera un nuovo asset capitalizzabile; in tali situazioni
> potrebbe essere valutata una classificazione CAPEX secondo le regole Finance. In ambito MMD sono
> presenti, ad esempio, SE707001.0001 Consulenza Informatica per applicazioni e SE707002.0001
> Consulenza Informatica per infrastrutture digitali. Le linee guida operative CAPEX/OPEX
> indicano che i costi di gestione ordinaria e i servizi ricorrenti sono generalmente OPEX…

**Cosa guardare dopo:** che **risponda** (il 21/10 alle 12:25 il turno non si chiudeva), **una
sola** risposta, i due codici `SE707001/SE707002`, e il rimando alle regole Finance.
Prova: `prove_uat/20261021_BASELINE_3_contabilizzazione.png`

---

## Caso 4 — Francese su RSSE, DUE TURNI (il caso fragile)

**Turno 1:** *Quels codes MMD existent pour l'entretien des installations mécaniques navales ?*

> MMD | …le code trouvé est : SE103003.0001 – Entretien des installations mécaniques navales
> Ce code n'est actuellement pas disponible dans la société RSSE ; il est référencé dans une autre
> entité juridique (PATR) et devrait donc être demandé pour utilisation dans RSSE.

**Turno 2:** *Il s'agit de la maintenance des moteurs et des systèmes de propulsion à bord des
navires.*

> MMD | Pour la maintenance des moteurs et des systèmes de propulsion à bord des navires, le code
> le plus pertinent identifié dans le catalogue est : SE103003.0001 – Entretien des installations
> mécaniques navales … Il n'est pas actuellement publié dans la société RSSE ; il est disponible
> dans l'entité juridique PATR et doit donc être demandé pour utilisation dans RSSE.

**Cosa guardare dopo, ed e' il caso piu' importante:**
- entrambi i turni **in francese**;
- **nessuna descrizione inglese con traduzione fra parentesi**;
- il **secondo turno risponde alla domanda nuova**, non a quella del primo;
- **non chiede "in quale lingua stai scrivendo"**;
- dice che il codice **non e' disponibile in RSSE**.

**Questo e' il caso che e' stato rotto e ripristinato piu' volte oggi.** Se dopo il cambio modello
salta qualcosa, molto probabilmente salta qui.
Prove: `prove_uat/20261021_BASELINE_4a_FR_turno1.png`, `…_4b_FR_turno2.png`

---

## Caso 5 — Francese su RSPA: **gia' difettoso oggi**

**Domanda** (societa' RSPA): *Quels codes existent pour l'entretien des équipements de
laboratoire ?*

> MMD | **Nessun articolo** relativo all'entretien des équipements de laboratoire **è stato trovato
> nella società RSPA** con i termini di ricerca forniti. **Non risultano codici disponibili** da
> proporre sulla base della ricerca effettuata.

**Attenzione: la risposta e' in ITALIANO a una domanda in francese.** Su RSSE lo stesso tipo di
domanda esce in francese, su RSPA no.

**Questo e' lo stato di fatto di oggi, non una regressione del cambio modello.** Va messo agli
atti proprio perche' dopo nessuno lo scambi per un effetto del nuovo modello.
Prova: `prove_uat/20261021_BASELINE_5_RSPA_francese.png`

---

## I tempi, che contano quanto le risposte

Durate del flusso `RINA - Search Items` nelle run di questa sessione:

| ora | durata |
|---|---|
| 17:02 | 00:00:54 |
| 17:16 | 00:00:39 |
| 17:17 | 00:00:06 |
| 17:22 | 00:00:31 |
| 17:26 | 00:00:26 |

**Riferimento: fra 6 e 54 secondi, nessun fallimento.** Il caso peggiore noto resta la ricerca su
RSPA con molti codici, intorno a **1m41s**, che supera l'attesa del client Copilot.

**Dopo il cambio modello vanno rimisurate**: siamo a ~1m01s sul caso standard contro un limite che
scatta verso 1m40s. **Un modello piu' lento ci riporta dentro il timeout**, e sarebbe un difetto
che si manifesta come "non risponde", non come "risponde male".

---

## Come si usa

1. Cambiare modello (vedi `20261021_ritiro_GPT55_piano.md`).
2. Rifare **questi sei casi, nello stesso ordine, in F&O**, non nel pannello di test.
3. Confrontare riga per riga con quanto sopra.
4. Rileggere le durate delle run.
5. Se qualcosa cambia, **il confronto dice subito se e' il modello o altro**: a parita' di
   istruzioni e flusso, la differenza e' il modello.
