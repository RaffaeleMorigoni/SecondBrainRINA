# Il "prima" del cambio modello — rilevato il 30/10/2026, ore 10:20-10:55

La baseline del 21/10 **non e' piu' un riferimento valido**: da allora abbiamo cambiato il filtro
del flusso e due blocchi di istruzioni. Confrontare il dopo-cambio con quella mescolerebbe le
variabili. Questo file e' il "prima" **allineato alle istruzioni e al flusso di oggi**, ancora su
**GPT-5.5 Chat**.

F&O, societa' **RSSE**, utenza `C_110912@rina.org`, **lingua di interfaccia `it`** (va sempre
dichiarata, vedi `20261030_test_lingua_interfaccia.md`), sessione Copilot nuova per ogni caso.

---

## Caso 1 — Ricerca in italiano: **OK**

*Devo acquistare un servizio di manutenzione per le attrezzature di laboratorio*

> MMD | SE153005.0001 Manutenzione altre apparecchiature specialistiche di laboratorio
> Per un servizio di manutenzione relativo ad attrezzature di laboratorio, questo e' l'articolo
> rilasciato nella societa' RSSE che risulta piu' pertinente. Sono disponibili anche voci piu'
> specifiche qualora l'attrezzatura rientri in una categoria particolare:
> SE153002.0001 Manutenzione impianti di laboratorio · SE153003.0001 Manutenzione microscopi
> elettronici · SE153004.0001 Taratura strumentazione di laboratorio · SE153005.0001 Manutenzione
> altre apparecchiature specialistiche di laboratorio
> Se si tratta di manutenzione generica … utilizzare SE153005.0001

Prefisso `MMD | ` presente, lingua italiana, un solo messaggio, codici corretti.
**Differenza dal 21/10**: apre con il codice piu' pertinente invece che con la frase, e i
quattro codici **non vanno a capo** (difetto di formato gia' noto in F&O, non una novita').

Prova: `prove_uat/20261030_PRIMA_cambio_modello_1_IT_manutenzione.png`

---

## Caso 4 — Francese, due turni: **INTERMITTENTE. Difetto aperto.**

Il caso piu' fragile della suite. Provato **due volte**, con esiti diversi.

### Turno 1 — OK in entrambe le prove

> MMD | SE103003.0001 Entretien des installations mécaniques navales
> Ce code MMD existe pour l'entretien des installations mécaniques navales, mais il n'est pas
> libéré dans la société RSSE. Il est actuellement disponible uniquement dans l'entité légale PATR.
> Pour des activités similaires déjà libérées dans RSSE, les codes disponibles sont :
> SE103001.0001 Entretien des installations mécaniques civiles · SE103002.0001 Entretien des
> installations mécaniques industrielles

Tutto in francese, nessuna parentesi di traduzione, dice che non e' rilasciato in RSSE.
**Meglio del 21/10**: aggiunge le alternative gia' disponibili in RSSE.

### Turno 2, prima prova — **ROTTO**

> **Qual e' la lingua in cui stai scrivendo il messaggio?**

**Chiede la lingua, e la chiede in italiano**, a una domanda in francese. Nessun prefisso
`MMD | `, nessuna risposta alla domanda.

E' il ritorno del difetto che risultava chiuso (vedi memoria `rina-non-chiedere-la-lingua`,
risolto con una riga nel blocco LANGUAGE del padre). Coerente con
`rina-currentuserrequest-non-aggiornato`: al secondo turno il contesto della lingua si perde.

Prova: `prove_uat/20261030_PRIMA_cambio_modello_4_FR_turno2_CHIEDE_LA_LINGUA.png`

### Turno 2, seconda prova — **OK**

> MMD | SE103003.0001 Entretien des installations mécaniques navales
> Étant donné qu'il s'agit de la maintenance des moteurs et des systèmes de propulsion à bord des
> navires, le code MMD le plus approprié … Il est actuellement disponible dans l'entité légale
> PATR et non libéré dans RSSE.

Tutto corretto: francese, risponde alla domanda **nuova**, prefisso presente, disponibilita' giusta.

Prova: `prove_uat/20261030_PRIMA_cambio_modello_4_FR_turno2_OK.png`

---

## Il punto che conta

**Questo difetto esiste PRIMA del cambio modello**, ed e' **intermittente: una volta su due**.
Va messo agli atti adesso, perche' se riappare dopo il cambio **non e' colpa del modello nuovo**.
E' esattamente il motivo per cui valeva la pena rifare il "prima" invece di fidarsi di quello
del 21/10.

Da riprendere **dopo** il cambio modello e dopo la suite, non ora: una cosa alla volta.
