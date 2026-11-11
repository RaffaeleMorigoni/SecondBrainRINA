# Commenti DevOps da pubblicare — 16/10/2026

## Testo comune (lingua) — per il task della segnalazione lingua di Emanuela

Risolto. La causa non era una sola.

1. **Le descrizioni arrivavano sempre in italiano.** Il flusso di ricerca restituiva il campo
   italiano dell'anagrafica qualunque fosse la lingua della conversazione. Ora legge le
   traduzioni ufficiali gia' presenti in Dynamics (entita' ProductTranslations): italiano,
   inglese, francese, portoghese, spagnolo e cinese. Non sono traduzioni automatiche, sono i
   testi caricati da RINA. Copertura verificata su 15 codici MMD nuovi: 15 su 15 hanno
   francese, portoghese e cinese.

2. **L'orchestratore aveva il divieto esplicito di tradurre.** Nelle sue istruzioni il blocco
   PROCUREMENT RESULTS diceva di copiare la risposta dell'agente di ricerca "as it is" e
   "never translate it". Per questo le frasi restavano in italiano anche quando la domanda era
   in un'altra lingua. Ora la regola e' "write it in the user's language, never reformat it or
   drop its MMD label": si traduce, ma il formato resta quello concordato.

Esito dei test (stessa domanda, chat pulita, societa' RSSE):

| lingua | frasi dell'agente | descrizioni |
|---|---|---|
| italiano | ok | ok |
| francese | ok | ok |
| portoghese | ok | ok |
| cinese | ok | ok |
| inglese | italiano | ok (inglese) |
| spagnolo | italiano | italiano |

Screenshot allegati, uno per lingua.

**Due cose restano aperte e non dipendono dall'agente:**

- **Spagnolo**: in anagrafica diversi codici hanno la traduzione `es` vuota o scritta in inglese
  (es. SE302501.0001 ha `es` vuoto). Finche' il dato non viene completato, un utente spagnolo
  vedra' l'italiano. Da girare a chi gestisce l'anagrafica.
- **Inglese**: le descrizioni escono correttamente in inglese, ma le frasi di contorno restano
  italiane. Unico caso rimasto, in corso di analisi.

**Nota sui dati**: alcuni codici hanno la descrizione *italiana* gia' scritta in inglese (es.
SE302501.0001 = "Software projects development"). Non e' un difetto dell'agente: spiega le
risposte con voci in lingue miste viste nei test.

## 2332 — company della ricerca
Done. La ricerca usa la legal entity della sessione F&O e non piu' la societa' predefinita
dell'utente. Verificato con la stessa domanda su due societa': su RSSE restituisce i codici
SE7070xx di RINA Services, su RSPA no. Corretto anche il parametro Cross Company del flusso, che
leggeva solo la societa' della connessione: era la causa dei "non trova niente" su RSSE.

## 2334 — formattazione
Resta in Ready for testing. Il formato grezzo (`| Goods | category ... | unit LUMPSUM`) e' stato
eliminato nel flusso: l'output non e' piu' prosa ma dati strutturati. L'elenco esce uno per riga
con la descrizione accanto al codice.

## 2333 — doppio messaggio
Da riaprire: il comportamento si e' ripresentato durante i test.

## 2324 e 2328 — visibilita' del disponibile
Aggiornare con l'esito: l'agente mostra le due liste (disponibile in societa' / da richiedere) e
lascia all'utente la scelta del codice.

---

# Stato al 19/10/2026 — cosa e' stato effettivamente pubblicato

Il testo comune qui sopra e' stato pubblicato il 16/10. Il 19/10 i quattro task sono stati
rieseguiti con le **domande esatte delle segnalazioni** e commentati di nuovo. Dettaglio completo
in `20261019_stato_UAT.md`.

| task | commento 19/10 | allegati | stato |
|---|---|---|---|
| 2395 portoghese | pubblicato | 2 | **Done** |
| 2396 francese | pubblicato | 1 | **Done** |
| 2397 cinese | pubblicato | 1 | **Done** |
| 2404 inglese | pubblicato, **con rettifica** | 3 | **Ready for testing** |
| 2416 spagnolo (nuovo) | creato, figlio di 2394 | 1 | In Progress |

**Due scostamenti dal piano**, entrambi perche' «Resolved» non esiste in questo processo: per gli
Impediment l'unica chiusura e' **Done**; il 2404 e' un Task ed e' stato lasciato in
**Ready for testing** perche' contiene una rettifica che Emanuela deve riverificare.

## La rettifica sul 2404 — da tenere presente

Il commento del 16/10 diceva che l'inglese dipendeva dalla **lingua di interfaccia F&O**. **Era
sbagliato**: misura fatta prima che la correzione alle istruzioni dell'orchestratore facesse
effetto. Il 19/10, con interfaccia in italiano, l'inglese esce interamente in inglese, tre prove
su tre. La rettifica e' stata pubblicata sul task. Non usare piu' la spiegazione del locale, ne'
con Emanuela ne' altrove.

## Gli altri quattro commenti di questo documento

I paragrafi 2332, 2334, 2333, 2324/2328 **non sono ancora stati pubblicati**: restano in attesa
del tuo OK.

---

# 21/10/2026 — pubblicati tutti e quattro

| task | tipo | cosa ho scritto | stato dopo |
|---|---|---|---|
| **2332** Filtro articoli per LE | Task | societa' della sessione F&O + Cross Company corretto, riprovato oggi su RSSE e RSPA | **Ready for testing** |
| **2334** Formattazione | Task | solo il *perche'*: tolto dal flusso il formato grezzo, l'agente riceve dati strutturati | Done (era gia' chiuso) |
| **2333** Doppio messaggio | Task | il dubbio del 16/10 e' rientrato: dieci prove oggi, sempre una risposta sola | Done |
| **2324** Presentazione proattiva | Task | **risposta a Chiara** | In Progress |

**Sul 2324 c'era una domanda in attesa da ieri.** Chiara aveva scritto: *"ieri al SAL hanno escluso
questo task, emerso dal Business ma rifiutato da IT. Possiamo toglierlo?"* — quindi **non andava
messo in Ready for testing**. Ho risposto che per me si puo' togliere, segnalando che il
comportamento c'e' gia' comunque, e che la decisione e' di lei e Andrea.

**Nota di metodo:** i commenti sono stati riscritti molto piu' corti di come erano preparati qui
sopra, in prima persona e senza tabelle. Per i prossimi: stesso registro.

---

# 28/10/2026 — 2495, pubblicato

**2495 — Problema Descrizione dell'articolo in italiano** (Task, figlio del PBI 2493
*28/10/2026 Test in Lingua*, aperto da Andrea Acerbi). Mi era gia' assegnato, **In Progress**:
stato lasciato invariato, il fix non e' ancora verificato a freddo.

Il task contiene **due segnalazioni diverse** nella stessa descrizione: l'errore di accesso di
Joao e la descrizione italiana vista da Nistor. Nel commento le ho tenute separate:

1. l'accesso era la **condivisione dell'agente**, non l'ambiente F&O — aggiunti Joao e Anamaria;
2. le lingue ci sono (copertura piena sugli MMD nuovi, manca solo lo spagnolo, che sta sul 2416);
   quello di Nistor succede quando la ricerca non parte o va in timeout e l'agente ripiega sui
   dati della pagina aperta. Stesso tema del **2418**;
3. oggi ho pubblicato la modifica alle istruzioni del padre, da **riprovare a freddo domani
   mattina in F&O** (vedi `20261028_fix_mancata_invocazione.md`).

Niente tabelle, quattro paragrafi, prima persona: stesso registro del 21/10.
