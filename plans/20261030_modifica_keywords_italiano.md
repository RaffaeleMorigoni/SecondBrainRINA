# Keywords in italiano: modifica pubblicata, ma il miglioramento non e' suo

30/10/2026, flusso `RINA - Search Items`, trigger `When an agent calls the flow`,
descrizione del parametro **`Keywords`**. Pubblicato alle 16:24.

## Cosa ho trovato prima di toccare

La descrizione diceva:

> Item code, or up to 4 single keywords separated by commas. Give one keyword to each distinct
> thing the user asked for **before adding any English translation**: equipment and materials are
> two different things. **Italian and English**, e.g. attrezzatura, materiali, laboratorio, equipment

E quella di `UserLanguage` dava gia' per scontata la traduzione:

> never from the keywords **you translated for the search**: a question written in Portuguese is
> pt even when you search in Italian

**Il disegno prevedeva gia' la traduzione, ma `Keywords` non la ordinava mai.** *"Italian and
English"* descrive un esempio, non impone una lingua. Con una domanda francese il modello non
stava disobbedendo — lo stesso schema del 2418 ([[rina-permesso-dati-pagina]]).

## Cosa ho scritto

> Item code, or up to 4 single keywords separated by commas. **Always write the keywords in
> Italian: the search runs on Italian product names, so a keyword in another language finds
> nothing. Translate the user's words into Italian yourself, whatever language the user wrote
> in.** Give one keyword to each distinct thing the user asked for before adding any English
> translation: equipment and materials are two different things. Italian first, English after,
> e.g. attrezzatura, materiali, laboratorio, equipment

504 caratteri su 1024. Save draft, Publish, stato `Published`.

## L'esito in F&O: due prove su due, riuscite

Stessa domanda delle 15:40, sessione nuova, RSSE, interfaccia italiana.

**Prova 1 (16:27)** — risposta ricca in francese con `SE152002.0001 — Manutenzione attrezzature
tecnologiche [maintenance des equipements technologiques]`, tipo di contabilizzazione, conto,
categoria di progetto.

**Prova 2 (16:36)** — elenco completo, formato corretto:
`SE105001.0001`, `SE152002.0001`, `SE153002.0001`, `SE153003.0001`, `SE153005.0001`,
`SE303002.0001`, tutti con descrizione francese, chiuso da *"Le code approprie depend donc de
l'equipement concerne."*

Dove prima usciva *"Aucun code MMD… n'a ete identifie"*.

## Ma la causa non e' la modifica, e va detto

In Power Automate, **le keyword passate sono rimaste francesi in tutte e quattro le chiamate**
successive alla pubblicazione: `entretien, equipement, maintenance` e `maintenance, equipements,
service`. **La descrizione nuova non ha cambiato cio' che il padre passa.**

E il confronto che conta e' questo:

| | 15:40 (prima) | 16:36 (dopo) |
|---|---|---|
| keywords | `maintenance, equipements, service` | `maintenance, equipements, service` |
| `Lista prodotti 1-4` | 1s, 0.6s, 1s, 1s | **22s, 17s, 22s, 17s** |
| totale | 7s | 42s |
| risultati | **nessuno** | **sei codici** |

**Input identico, esito opposto.** Non e' la lingua delle parole a fare la differenza: **il
difetto e' intermittente**. Le stesse parole francesi che alle 15:40 non trovavano niente alle
16:36 trovano sei codici.

Questo rimette in discussione anche la lettura di stamattina: il confronto FR/IT delle 15:40-15:41
sembrava provare la causa linguistica, ma era **un campione solo per lato**.

## Cosa lascio e cosa resta da fare

**La modifica resta pubblicata**: e' corretta a prescindere — rende esplicito cio' che
`UserLanguage` gia' presupponeva — e non ha prodotto effetti negativi. Ma **non attribuirle il
merito** finche' non e' dimostrato.

**Da fare domani a freddo**, in questo ordine:
1. **tre prove francesi consecutive**, sessione nuova ogni volta, annotando durata e numero di
   risultati: se oscillano, l'intermittenza e' confermata e il bersaglio diventa il 2494;
2. solo se le tre falliscono tutte, tornare sulla descrizione — e allora il punto giusto non e'
   l'input dello strumento ma le **istruzioni del padre**, che e' chi compone le keyword.

Una variabile alla volta: oggi e' gia' cambiato il modello ([[rina-ritiro-gpt55-30-ottobre]]).

Prove: `prove_uat/20261030_FR_dopo_modifica_sei_codici.png`,
`prove_uat/20261030_PA_keywords_ancora_francesi_ma_trova.png`.

---

# LE TRE PROVE, FATTE SUBITO — 2 su 5

Stessa domanda, sessione nuova ogni volta, RSSE, interfaccia italiana, nell'arco di 25 minuti.

| | ora | esito | run del flusso |
|---|---|---|---|
| A | 16:27 | **riuscita** — `SE152002.0001` con dettagli | 26s + 4s |
| B | 16:36 | **riuscita** — sei codici in elenco | **42s** + 5s |
| 1 | 16:45 | **fallita** — *"Aucun code MMD… n'a ete trouve"* | **20s** + 3s |
| 2 | 16:47 | **fallita** — *"Les resultats disponibles concernent d'autres types de services"* | **20s** + 2s + 4s |
| 3 | 16:49 | **fallita, e in italiano** | **23s** + 4s + 3s + 4s |

**Due successi, tre fallimenti, input identico.** L'intermittenza e' confermata, e la modifica
alla descrizione non la governa.

## Il segnale nei tempi

Il numero che separa i due gruppi non e' la lingua: e' **quanto dura la prima run**.

- riuscite: **26s e 42s**
- fallite: **20s, 20s, 23s**

Sopra i 25 secondi trova, sotto i 25 non trova. E nelle fallite il padre **ritenta**: due, tre,
quattro chiamate di seguito, tutte brevissime (2-4s), tutte a vuoto.

La lettura piu' semplice che regge i dati: **le query tornano parziali o vuote quando il tempo
disponibile non basta**, e il flusso le tratta come "nessun risultato" invece che come errore.
E' il 2494 ([[rina-search-items-query-sequenziali]]), non un problema di lingua. Con quattro
query sequenziali da 17-22s ciascuna il margine e' sempre sul filo.

## La prova 2 e' quella che lo dimostra meglio

> *"Les resultats disponibles concernent d'autres types de services et ne sont pas suffisamment
> pertinents pour recommander un code."*

**La ricerca aveva trovato qualcosa.** Non e' il caso "zero risultati": sono arrivati risultati
parziali, e l'agente li ha scartati perche' non pertinenti. Un troncamento, non un vuoto.

## E un difetto in piu', trovato per caso

La **prova 3 ha risposto in italiano** a una domanda francese, con entrambe le sezioni
(*Articoli disponibili nella tua societa'* / *Articoli presenti in Dynamics ma da richiedere*).
Quando la ricerca non trova, la risposta perde anche la lingua: il padre ricade sulla prosa
italiana ([[rina-padre-copia-non-traduce]]). I due difetti si sommano proprio nel caso peggiore.

## Conclusione

**Il bersaglio e' il 2494, non la lingua.** Le keyword francesi funzionano quando la ricerca ha
tempo di completarsi. Prima di qualunque altra modifica al prompt o alle descrizioni va chiuso
il collo di bottiglia delle query.

La descrizione nuova di `Keywords` resta pubblicata — e' corretta e non fa danno — ma **non
risolve niente** e non va presentata come un fix.
