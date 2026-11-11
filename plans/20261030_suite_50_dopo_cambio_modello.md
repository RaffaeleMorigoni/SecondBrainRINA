# Suite da 50 casi dopo il cambio modello — 30/10/2026

Run lanciata alle **11:55** sull'agente standalone `RINA Procurement & MMD Assistant`
(`dfd051a3…`), test set **Evaluate RINA Procurement & MMD Assistant** (50 casi),
profilo `C_110912@rina.org`, connessione Fin & Ops attiva.
Nome run: `Evaluate RINA Procurement & MMD Assistant 261030_1155`.

**Unica variabile rispetto alle run storiche: il modello.** Istruzioni e flusso sono quelli di
ieri sera (il filtro sul codice e le keyword italiane), nessuna modifica ai prompt oggi.

| | 08/10 mattina | 08/10 sera | 09/10 ripristino | 09/10 + categorie | **30/10 Chat Latest** |
|---|---|---|---|---|---|
| modello | GPT-5.5 | GPT-5.5 | GPT-5.5 | GPT-5.5 | **GPT Chat Latest** |
| Answer quality | 80% | 82% | 78% | 80% | da misurare |
| Meaning match | 64% | 52% | 60% | 52% | da misurare |
| sezione "da richiedere" | - | 1/50 | 7/50 | 11/50 | da contare |
| prefisso `MMD \| ` | - | 43/50 | 46/50 | 43/50 | da contare |

## Come va letta (regole gia' imparate, da non rifare)

- **I punteggi aggregati oscillano in banda** (78-82 e 52-60) senza seguire le modifiche:
  non usarli per decidere.
- **Guardare la distribuzione, non la media.** Un blocco di fallimenti tutti allo stesso valore
  (es. tutti a 50) e' la firma di un artefatto di misura, non di una regressione.
- **Un caso che fallisce una volta e' rumore**: serve che fallisca in due run consecutive.
- **Un caso andato in `FlowActionTimedOut` nella suite va rieseguito da solo** prima di aprire
  un difetto: sotto carico i tempi si allungano.
- Gli indicatori che contano davvero sono quelli **strutturali**: sezione "da richiedere",
  prefisso `MMD | `, codici legacy esposti.

## Cosa mi aspetto, visto quello che ho misurato stamattina in F&O

- **prefisso**: in F&O Chat Latest lo metteva **una volta sola e correttamente** → dovrebbe
  salire rispetto a 43-46;
- **elenco a capo**: migliorato, ma la suite non lo misura;
- **secondo turno**: la suite e' a risposta singola, quindi **non intercetta** il difetto che ho
  visto stamattina (dice "nessun codice trovato" al follow-up). Resta da verificare a mano.

## Esito: **Answer quality 68%, Meaning match 42%, 50/50 in 24:52**

| | 08/10 mattina | 08/10 sera | 09/10 ripristino | 09/10 + categorie | **30/10 Chat Latest** |
|---|---|---|---|---|---|
| modello | GPT-5.5 | GPT-5.5 | GPT-5.5 | GPT-5.5 | **GPT Chat Latest** |
| Answer quality | 80% | 82% | 78% | 80% | **68%** |
| Meaning match | 64% | 52% | 60% | 52% | **42%** |

**Entrambi gli indicatori sono sotto il minimo storico** (76 e 52). Non e' l'oscillazione in
banda gia' documentata: e' fuori banda su tutti e due i metodi contemporaneamente.

## Ma la causa non e' il contenuto: e' il fallback

**Otto dei dodici fallimenti sono la stessa cosa**: *"Mi dispiace, non so come posso aiutarti.
Puoi pro..."*, cioe' il fallback generico, non una risposta sbagliata.

| caso | punteggio |
|---|---|
| Ricerca un codice per lo sviluppo di soluzioni… | 0 |
| MMD code per una prestazione professionale… | 0 |
| devo effettuare un acquisto che prima caricavo… | 0 |
| Forniscimi il codice materiale per prodotti… | 0 |
| laboratorio | 0 |
| MMD code collegato al conto di contabilita'… | 25 |
| [BRD 1.1] Scegli tu la contabilizzazione giusta | 25 |
| [BRD 3] Compilami il draft della richiesta | 25 |

Gli altri quattro sono **risposte vere ma incomplete**, tutte a 50: polizza assicurativa,
[BRD 1.1] creazione richiesta, [BRD 1.3] reagenti chimici, [Schema decisione 1] MA101004.

## La controprova in F&O: il caso che falliva risponde

Preso il primo dei casi a 0 e riproposto in F&O, RSSE, sessione nuova:

```
MMD code per una prestazione professionale in ambito 9001
MMD | PS101002.0002 - ISO 9001
```

**`PS101002.0002` e' esattamente l'atteso della suite.** Il caso che la valutazione segna 0 per
mancata risposta, nel canale che usano i tester **risponde, e risponde giusto**.

Prova: `prove_uat/20261030_caso_fallito_nella_suite_risponde_in_FO.png`

Conferma [[rina-fallback-generico-orchestratore]]: il "Mi dispiace" e' un artefatto del canale
di valutazione. **I punteggi vanno letti al netto di quegli otto casi.**

## Cosa resta vero comunque

- **Il prefisso `MMD | ` c'e' su tutte le risposte vere** osservate nella griglia: nessuna delle
  risposte buone ne e' priva. E' il miglior dato di forma di sempre, coerente con quanto misurato
  in F&O stamattina.
- **Otto fallback su 50 sono piu' dei quattro-sei delle run storiche.** Non e' dimostrato che
  dipenda dal modello, ma e' il numero da riguardare nella prossima run.
- **Le risposte sono molto piu' asciutte.** `PS101002.0002 - ISO 9001` e' corretto ma spoglio:
  niente sezioni, niente contesto. Lo stesso tratto visto in F&O stamattina.
- **Il secondo turno non e' misurato**: la suite e' a risposta singola, quindi il difetto visto
  stamattina (*"aucun MMD trouve"* al follow-up) **non compare qui**. Va verificato a mano.

## Giudizio

Il cambio modello **non ha prodotto regressioni di contenuto dimostrate**: i codici che escono
sono quelli attesi. Ha pero' **asciugato le risposte** e, forse, aumentato i fallback.

**Da fare prima di decidere**: una seconda run, perche' un solo campione non distingue il segnale
dal rumore - e' la regola che il 06/10 e' costata tre correzioni inutili. Se anche la seconda
sta a 68/42 con otto fallback, allora il dato e' reale.

---

# SECONDA RUN — `261030_1514`, confermata

Lanciata alle 15:14, stesso test set, stesso profilo `C_110912@rina.org`, connessione
Fin & Ops verde. **50/50 in 24:59.**

| | prima run `261030_1155` | **seconda run `261030_1514`** |
|---|---|---|
| Answer quality | 68% | **64%** |
| Meaning match | 42% | **36%** |
| casi falliti | 12 | **16** |
| di cui fallback *"Mi dispiace"* | 8 | **8** |
| di cui risposte vere ma a 50 | 4 | **7** |

## Il dato non e' rumore

Due run consecutive sotto la banda storica (76-82 / 52-64) su **entrambi** i metodi.
La regola del 06/10 e' soddisfatta: il campione si ripete.

## Ma e' lo stesso difetto gia' documentato, e colpisce gli stessi casi

Gli **otto fallback sono esattamente gli stessi otto casi** della run delle 11:55:

a punteggio 0 — *vorrei acquistare un servizio di consulenza*, *Ricerca un codice per lo
sviluppo di soluzioni*, *MMD code per una prestazione professionale*, *Forniscimi il codice
materiale per prodotti*, *laboratorio*;
a punteggio 25 — *MMD code collegato al conto di contabilizzazione*, *[BRD 1.1] Puoi creare tu
la richiesta d'acquisto*, *[BRD 1.1] Scegli tu la contabilizzazione*.

Stessi casi, due volte di fila: **e' sistematico, non casuale**. Ed e' lo stesso elenco su cui
stamattina la controprova in F&O ha risposto correttamente
(`MMD | PS101002.0002 - ISO 9001`, che e' l'atteso della suite).
Vedi [[rina-fallback-generico-orchestratore]].

## Cosa peggiora davvero

I casi a **50** passano da 4 a 7. Sono risposte vere, con il codice giusto, che perdono punti
perche' sono **troppo asciutte**: il Meaning match confronta il senso, e una risposta ridotta al
solo codice non copre l'atteso. E' lo stesso tratto osservato stamattina in F&O.

**Questo si', e' attribuibile al cambio modello.** Non il contenuto: la ricchezza della risposta.

## Giudizio

Il cambio modello **non ha introdotto errori di contenuto**: i codici che escono restano quelli
attesi. Ha pero' asciugato le risposte quanto basta a far perdere punti sul Meaning match, e i
numeri complessivi non sono confrontabili con lo storico perche' otto casi su cinquanta non
misurano l'agente ma l'artefatto della suite.

**Da non fare**: cambiare di nuovo modello per inseguire il punteggio. Chat Latest resta l'unico
dei tre provati con il formato corretto in F&O, che e' il canale dei tester.

**Da fare**: se si vuole recuperare ricchezza, agire sul template della risposta
([[rina-template-risposta-ricerca]]), non sul modello. Una variabile alla volta.
