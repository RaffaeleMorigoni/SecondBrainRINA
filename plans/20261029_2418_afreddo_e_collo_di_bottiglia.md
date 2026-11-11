# 29/10 — prova a freddo del 2418: il fix ha preso, ma la run muore sul timeout

Prova eseguita in **F&O**, non nel pannello di test, con la riga a video: la condizione che
innescava il difetto.

- societa' **RSSE**, *Dettagli prodotto rilasciato*, riga `EX102002.0001` selezionata;
- interfaccia F&O **in italiano**, domanda **in inglese**: `item number EX102002.0001`;
- ore **14:27**, il giorno dopo la pubblicazione (fix del 28/10 ore 17:45).

## Esito: il controllo che contava e' passato

**Il flusso e' partito.** Run `Oct 29, 02:27 PM` nello storico di `RINA - Search Items`.
Prima del fix l'agente rispondeva **subito** con il `Product name` della griglia e **nessuna run**.
Il permesso di troppo nel blocco SCOPE era davvero la causa: il 2418 e' chiuso sul suo difetto.

**Ma la run e' andata in `TimedOut` a `00:02:00`** e l'utente ha visto
*"Il bot non puo' parlare per al momento. Codice errore: FlowActionTimedOut"*.
Il difetto non e' piu' "risponde la cosa sbagliata", e' "non risponde". Per il tester e' comunque
un fallimento.

---

## Dove si inchioda, con i numeri della run

| azione | esito |
|---|---|
| `Serve la societa` | 0.1s |
| `Filtra parole` | 0s — **una sola parola: `EX102002.0001`** |
| `Lista prodotti 3` | **21s**, completata |
| `Lista prodotti 1`, `2`, `4` | **mai tornate**, `1 retries occurred` su Lista prodotti 1 |

Il banner del portale e' esplicito: *"Your flow timed out. Flows running in **Express mode** must
complete within 2 minutes."* Express mode risulta **Enabled** (riattivato il 21/10).

### La causa, letta sull'input della run

```
$filter: AVY0587_PurchaseNature eq ...'Hybrid' and (
           ProductSearchName eq '*EX102002.0001*' or ProductNumber eq 'EX102002.0001*' )
$top: 40   $select: ProductNumber
```

**Il wildcard sta anche in testa**: `'*...*'` e' un *contiene*, non un *inizia con*, e su un campo
non indicizzato costringe alla **scansione dell'intera tabella**. E' la stessa famiglia di
problema gia' vista l'08/10 su `Lista nature` (filtro su campo custom, 50s su 96).

**Rettifica a quanto avevo scritto stamattina**: avevo letto nel code view il ramo con
`ProductSearchName eq 'parola*'` e concluso "si cerca solo per prima parola". Vale per quel ramo;
`Lista prodotti 1` usa invece `'*parola*'`. Le due forme convivono nello stesso filtro, a seconda
dell'indice della parola e della sua lunghezza. Da rileggere per intero prima di toccarlo.

---

## La mossa che propongo, e perche' questa

La domanda era **solo un codice**. Cercare quel codice **dentro le descrizioni** non serve a
niente: basta `ProductNumber`. Il `'*...*'` su `ProductSearchName` e' lavoro buttato che costa una
scansione piena, e in questo caso e' anche l'unica ragione del timeout.

Intervento: quando la parola ha la forma di un codice MMD, interrogare **solo `ProductNumber`**.
E' chirurgico, misurabile sulla stessa domanda, e non tocca la ricerca per descrizione.

**Poi** — e separatamente — restano in coda, nell'ordine gia' deciso:
1. parole chiave in italiano, per il caso di `plans/20261029_ricerca_in_inglese_non_trova.md`;
2. il wildcard in testa sulle ricerche per descrizione, che e' il 2494 vero e proprio;
3. Express mode: il tetto di 2 minuti e' suo. Il 15/10 disattivarlo aveva risolto una ricerca
   ampia (TimedOut 2:00 → 00:51 Succeeded); il 21/10 e' stato riattivato perche' taglia ~23s.
   Da rimettere in discussione **con misure**, non a sensazione.

Una modifica alla volta.

## Contesto: oggi il flusso non e' sempre lento

Run di oggi prima della prova: 11:49 `1m09`, 11:51 `23s`, 11:53 `10s`, 11:54 `8s`, 11:57 `9s`,
12:01 `32s`, 12:03 `4s` — **tutte Succeeded**. Media dichiarata 42s. Il timeout non e'
sistematico: dipende dalla domanda, ed e' per questo che sfugge ai test.

Prova: `prove_uat/20261029_2418_afreddo_flusso_invocato_ma_timeout.png`.

---

# Fix applicato e verificato — 29/10 ore 15:01

Pubblicato alle **15:01**. Modificate le quattro `Lista prodotti`, ciascuna nel solo campo
*Filter Query*. Punto di ripristino: `backup/20261029_ROLLBACK_lista_prodotti_filtro_codice.md`.

Tre cose in una riscrittura:

1. **ramo nuovo**: se la parola contiene un punto la si tratta come codice e si interroga **solo
   `ProductNumber`**. Sparisce il `ProductSearchName eq '*codice*'`, il *contiene* con wildcard in
   testa che costringeva alla scansione piena;
2. **`#none#` → `ZZNONEZZ`**: erano gli ultimi quattro campi col debito del 05/10;
3. **`decodeUriComponent('%27')` → apostrofo raddoppiato**: stesso risultato, espressione da ~900
   a ~710 caratteri — serviva anche per poterla digitare senza far scadere il campo.

La ricerca per descrizione **non e' stata toccata**: i rami "parola corta" e "parola lunga"
restano identici, wildcard in testa compreso.

## Misure, stessa domanda e stesse condizioni

Provato prima sull'anagrafica: `ProductNumber eq 'EX102002.0001*'` con il filtro nature risponde
in **1,1 secondi** e restituisce il codice. Poi in F&O, RSSE, riga a video, `item number
EX102002.0001`:

| | run | esito |
|---|---|---|
| prima | `Oct 29, 02:27 PM` | **00:02:00 TimedOut**, "Il bot non puo' parlare per al momento" |
| dopo | `Oct 29, 03:05 PM` | **00:00:44 Succeeded**, scheda del codice a video |

Risposta ottenuta: `MMD | EX102002.0001 Deposito cauzionale`, con tipo, categoria `AdvExp`, unita'
di misura `LUMPSUM`, prezzo registrato e societa' `RSSE`.
Prova: `prove_uat/20261029_dopo_fix_codice_risposta_in_44s.png`.

## Quello che questa prova NON chiude

- **La domanda era in inglese e le etichette escono in italiano** (`Tipo`, `Unita' di misura`,
  `Societa'`). E' il tema lingua, separato, e ora si vede meglio di prima perche' prima non
  arrivava nessuna risposta.
- **44 secondi restano tanti** per una ricerca per chiave: il codice lo trova in 1,1s, il resto e'
  tutto quello che viene dopo. Il margine sul tetto dei 2 minuti c'e', ma non e' largo.
- Il fix vale **solo per le domande che contengono un codice**. Le ricerche per descrizione hanno
  ancora il wildcard in testa: e' il 2494 vero e proprio, prossimo in coda.
