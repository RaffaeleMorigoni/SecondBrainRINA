---
name: rina-ricerca-francese-non-trova
description: "In francese la ricerca a volte non trova nulla e a volte si: stesse keyword, esito opposto. Il difetto e' INTERMITTENTE, non linguistico - la prova del 30/10 era un campione solo per lato"
metadata:
  type: project
---

**Provato il 30/10/2026 in F&O**, RSSE, una sola sessione, due turni, **la stessa domanda**
nelle due lingue, modello GPT Chat Latest.

| | domanda | esito |
|---|---|---|
| turno 1, FR | *Je dois acheter un service de maintenance d'equipements…* | `MMD \| Aucun code MMD… n'a ete identifie` — **nessun `Operazione in corso`** |
| turno 2, IT | *Devo acquistare un servizio di manutenzione attrezzature…* | `MMD \| SE152002.0001 Manutenzione attrezzature tecnologiche` — flusso girato ~1 minuto |

**La causa e' la lingua delle parole chiave**, non il modello e non il turno: la ricerca
interroga `ProductSearchName`, che e' in italiano ([[rina-ricerca-parole-italiano-iniziaper]]).
`maintenance` non incontra `Manutenzione`. Le traduzioni esistono
([[rina-traduzioni-prodotto-esistono]]) ma **servono a scrivere la risposta, non a trovarla**.

**La forma francese e' perfetta**: prefisso corretto, francese pulito, nessuna prosa italiana.
Il lavoro sulla lingua ha funzionato — ed e' proprio questo che rende il difetto insidioso:
la risposta e' ben scritta e dice che il codice non c'e', mentre esiste.

**Come leggerlo in una segnalazione**: non e' *"l'agente non conosce i codici"*, e' **copertura
linguistica della ricerca**. Un tester francofono vedra' la prima cosa.

**Il secondo turno regge.** Era il sospetto aperto su Chat Latest (*"aucun MMD trouve"* al
follow-up, [[rina-ritiro-gpt55-30-ottobre]]): qui il secondo turno e' l'unico dei due che ha
trovato il codice. Non riprodotto in questa forma.

**Verificato in Power Automate, stesso giorno.** Il flusso **e' partito anche in francese** e
ha concluso `Succeeded`: la mia prima lettura (*"non e' partito"*) era dedotta dall'assenza
dell'indicatore a video ed era sbagliata.

| | FR 15:40 | IT 15:41 |
|---|---|---|
| `Filtra parole` in/out | `maintenance`, `equipements`, `service` | `manutenzione`, `attrezzature`, `servizio` |
| `Lista prodotti 1-4` | **1s, 0.6s, 1s, 1s** | **48s, 48s, 47s, 47s** |
| totale | 7s | 54s |

**Il padre passa le parole nella lingua dell'utente e nessuno le traduce.** Le query girano a
vuoto: 1s contro 48s non e' prestazione, e' una query che non trova niente da scandire.

**Si corregge fra l'estrazione e la query, non nel prompt**: o tradurre le parole in italiano
prima di cercare (tre parole), o cercare anche su `ProductTranslations` (quattro query da
allargare, ma sono gia' il collo di bottiglia del 2494).

Dettaglio e prove in `plans/20261030_secondo_turno_e_ricerca_francese.md`.


## RETTIFICA del 30/10 pomeriggio: e' intermittente, non linguistico

Dopo aver reso esplicito nella descrizione di `Keywords` l'ordine di tradurre in italiano
(pubblicato alle 16:24), due prove francesi su due **hanno trovato i codici**, la seconda con un
elenco completo di sei.

**Ma in Power Automate le keyword sono rimaste francesi in tutte le chiamate.** La modifica non
ha cambiato cio' che il padre passa. E il confronto a input identico e' questo:

| | 15:40 | 16:36 |
|---|---|---|
| keywords | `maintenance, equipements, service` | **le stesse** |
| `Lista prodotti 1-4` | 1s, 0.6s, 1s, 1s | 22s, 17s, 22s, 17s |
| risultati | nessuno | **sei codici** |

**Stesso input, esito opposto: il difetto e' intermittente.** La prova del mattino (FR fallisce,
IT riesce) sembrava dimostrare la causa linguistica ma era **un campione solo per lato** - lo
stesso errore che la regola del 06/10 vuole evitare.

**Regola che ne esce**: prima di attribuire un fallimento alla lingua, alla societa' o al
modello, **ripetere lo stesso caso almeno tre volte**. Qui l'intermittenza punta al 2494
([[rina-search-items-query-sequenziali]]), non alla copertura linguistica.

La descrizione nuova resta pubblicata perche' e' corretta in se', ma **non attribuirle il
merito**. Dettaglio in `plans/20261030_modifica_keywords_italiano.md`.

## Le tre prove: 2 riuscite su 5, e il segnale sta nei tempi

Stessa domanda, sessione nuova ogni volta, 25 minuti, 30/10 pomeriggio.

| esito | durata della prima run |
|---|---|
| riuscita (dettagli) | 26s |
| riuscita (sei codici) | **42s** |
| fallita | **20s** |
| fallita | **20s** |
| fallita, e in italiano | **23s** |

**Sopra i 25 secondi trova, sotto non trova.** Non e' la lingua: e' quanto dura la query.
Nelle fallite il padre ritenta due, tre, quattro volte, sempre con run da 2-4s, sempre a vuoto.

**La prova che lo chiude**: una delle fallite dice *"Les resultats disponibles concernent
d'autres types de services"* - **la ricerca aveva trovato**, i risultati erano parziali e
l'agente li ha scartati. Troncamento, non vuoto.

**Difetto in piu'**: quando non trova, la risposta **perde anche la lingua** e torna italiana
([[rina-padre-copia-non-traduce]]). I due difetti si sommano nel caso peggiore.

**Il bersaglio e' il 2494** ([[rina-search-items-query-sequenziali]]): quattro query sequenziali
da 17-22s lasciano un margine sempre sul filo. Finche' non si chiude quello, ogni modifica al
prompt o alle descrizioni misura rumore.
