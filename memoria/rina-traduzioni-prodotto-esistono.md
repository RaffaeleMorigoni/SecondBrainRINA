---
name: rina-traduzioni-prodotto-esistono
description: in F&O esiste l'entita ProductTranslations con le descrizioni MMD gia tradotte in 7 lingue, copertura piena sui codici nuovi
metadata:
  type: reference
---

Scoperto il 16/10/2026. Entita OData **`ProductTranslations`** (non "ReleasedProductTranslations",
che non esiste):
`/data/ProductTranslations?$filter=ProductNumber eq 'SE152002.0001'&$select=LanguageId,ProductName&cross-company=true`

Campi: `ProductNumber`, `LanguageId`, `ProductName`, `Description`, `Product`.

Lingue presenti per un codice nuovo: **it, pt-br, en-gb, en-us, fr, es, zh-hans** — cioe' tutte
quelle dei test di Emanuela. Esempio SE152002.0001:
- it "Manutenzione attrezzature tecnologiche"
- fr "Entretien des equipements technologiques"
- pt-br "Manutencao de equipamentos tecnologicos"
- zh-hans "技术设备维护"

**Copertura**: campione di 15 codici nuovi realmente restituiti dall'agente (SE*, ME*, PS*) ->
15 su 15 hanno fr + pt-br + zh-hans. Totali per lingua: it 81.619, pt-br 12.504, zh-hans 10.112,
fr 6.342 (l'italiano e' piu alto perche' include tutto il legacy Np*/pbus.*).

**Perche' conta**: risolve [[rina-lingua-elenco-vs-prosa]] senza traduzione automatica. Se il
flusso restituisce la descrizione gia nella lingua dell'utente, l'elenco copiato esce nella
lingua giusta e sono dati ufficiali RINA, non una traduzione inventata.

`startswith()` NON e' supportato nel $filter di questa entita; usare uguaglianze in or.

## Stato al 16/10/2026 notte - catena costruita, manca il passaggio della lingua

Nel flusso e' pubblicata tutta la catena (vedi `ROLLBACK_search_items_traduzioni.md`):
input `text_3/UserLanguage` + Clausole traduzioni -> Lista traduzioni -> Coppie traduzioni ->
Mappa traduzioni -> Seleziona articoli usa la descrizione tradotta con ripiego su SearchName.

**Blocco**: lo strumento in Copilot Studio non passa `text_3`. Provato dal run del 16/10 14:00,
il trigger riceve solo text/text_1/text_2. La sezione **Inputs della pagina dello strumento non
carica piu'** (vuota anche dopo ricaricamenti e dopo il pulsante di refresh dello schema): non si
riesce ad aggiungere l'input dalla UI.

**Nessuna regressione**: con text_3 vuoto il flusso ripiega su 'it', cioe' il comportamento
precedente. Verificato in italiano il 16/10.

**Via d'uscita da provare**: ricreare lo strumento dal flusso (il nuovo leggerebbe lo schema a 4
input), copiando prima la descrizione da 708 caratteri del vecchio, poi spegnere il vecchio.

## 16/10/2026 - COMPLETATO e pubblicato

Lo strumento vecchio non caricava piu' la sezione Inputs (bug della UI). Risolto **ricreando lo
strumento** dal flusso: il nuovo (b61db46a-5279-4f67-a415-798a64644b36) vede tutti e 4 gli input.
Il vecchio (77b79a2d-9418-4e55-bc17-a90c1ed7f547) e' stato spento, non cancellato.
Configurazione salvata in `strumento_search_items_config.md`.
Nota: alla creazione lo strumento mostra solo gli input OBBLIGATORI; quelli opzionali
(CompanyId, UserLanguage) vanno aggiunti con "Add input".

**Esito dei 3 casi francesi che prima fallivano:**
1. manutenzione attrezzature -> descrizioni francesi dall'anagrafica, prosa francese: pieno
2. microscopi elettronici -> descrizioni francesi, prosa dell'agente ancora italiana
3. formazione personale -> descrizioni inglesi con traduzione francese fra parentesi, prosa italiana

I **dati** ora escono nella lingua giusta in tutti i casi. La **prosa dell'agente** (le frasi di
contorno) resta instabile: 1 su 3. Il problema non e' piu' "risponde in italiano", ma
"accompagna dati tradotti con frasi italiane".

**Attenzione sui dati**: alcuni codici hanno il campo `it` gia' scritto in inglese (es.
SE302501.0001 it = "Software projects development"). Non e' un difetto dell'agente: e' come sono
registrati in anagrafica. Spiega la segnalazione di Emanuela del 16/10 sulle voci miste.

## 19/10/2026 - lo spagnolo e' l'unico buco, ed e' enorme (DevOps 2416)

Conteggi su `/data/ProductTranslations/$count?cross-company=true&$filter=...`:

| lingua | righe | di cui ProductName vuoto |
|---|---|---|
| it | 81.619 | - |
| en-gb | 80.881 | 1 |
| pt-br | 12.504 | 0 |
| zh-hans | 10.112 | 9 |
| fr | 6.342 | 9 |
| **es** | **1.897** | **1.288** |

Lo spagnolo resta con **609 descrizioni utilizzabili**, contro le 6.333 del francese che e' la
penultima. Le righe `es` esistono ma sono valorizzate a **stringa vuota**: non e' un problema di
righe mancanti, e per questo il flusso ripiega su SearchName (italiano). In piu' alcune `es`
contengono testo inglese (SE707004.0001 es = "IT consulting other").

**Come usarlo**: lo spagnolo non e' un difetto dell'agente e non si risolve da codice. Aperto
l'Impediment DevOps **2416** (figlio di 2394) per far popolare l'anagrafica. Dove il dato c'e',
viene usato: PS201040.0001 esce correttamente in spagnolo.

Il conteggio si fa cosi' (dalla console del browser, sessione F&O autenticata):
`fetch(base+encodeURIComponent("LanguageId eq 'es' and ProductName eq ''"),{headers:{Accept:'text/plain'}})`
con base = `https://<host>/data/ProductTranslations/$count?cross-company=true&$filter=`.

## 21/10/2026 - BUCO: il ramo "da richiedere" non passa dalla mappa traduzioni (DevOps 2419)

La catena delle traduzioni costruita il 16/10 e' stata applicata **solo al ramo dei disponibili**.

| azione | ramo | descrizione usata |
|---|---|---|
| `Seleziona articoli` | disponibili nella societa' | `Mappa traduzioni` con ripiego su `SearchName` — **tradotta** |
| `Seleziona non disponibili` | da richiedere / altre LE | `item()?['SearchName']` secco — **sempre italiana** |

**Prova incrociata, lo stesso codice in due task:** `SE354503.0001`
- **2418** su RSSE, dove e' disponibile: «Événements : Foires, Lieux et Organisation d'événements» → francese
- **2419** su RSPA, dove non e' disponibile: «Eventi: Fiere, Location, Preparazione Eventi» → italiano

Stesso articolo, stessa domanda in francese, due rami, due esiti. Non e' un problema di dato:
la traduzione `fr` esiste, e' il ramo che non la consulta.

**Correzione**: in `Seleziona non disponibili` sostituire `item()?['SearchName']` con la stessa
espressione di `Seleziona articoli`. Una riga. Attenzione: quel Select produce anche il formato
con le pipe (`codice | descrizione | Service | category X`), che e' il formato grezzo gia'
eliminato altrove — conviene sistemare le due cose insieme.

**Il titolo del 2419 e' fuorviante**: dice "risposta in Italiano" ma la prosa e' francese
corretta, in italiano c'e' solo la descrizione dell'articolo.

**Lezione sui miei test del 19/10**: avevo provato solo il **primo turno**, solo su **RSSE** e
solo su articoli **disponibili**. Le tre dimensioni scoperte da Chiara — secondo turno, altra
societa', ramo da richiedere — erano tutte fuori dalla mia copertura. Vedi
[[rina-suite-regressione]]: la suite lingua va estesa a questi casi.

### 21/10 - corretto e verificato

Catena dedicata nel ramo A, fra `Filtra non disponibili` e `Seleziona non disponibili`:
`Clausole traduzioni LE` -> `Lista traduzioni LE` (ProductTranslations) -> `Coppie traduzioni LE`
-> `Mappa traduzioni LE`. `Lista traduzioni` del ramo B **non e' stata toccata**, per non
appesantire la query che sta sul percorso critico del timeout.

**Verifica sul dato**, run 21/10 11:15 (1m 18s, 35 azioni), domanda in francese su RSSE.
`otherlegalentities` ora contiene:
`SE253002.0001 | Services de surveillance non armée | Service | ... released in FOUS`
Prima sarebbe uscito "Servizi di sorveglianza non armata".

**Resta da sistemare**: lo stesso codice compare una volta per ogni societa' in cui e' rilasciato
(5 righe identiche per FOUS, PATR, FOEE, FOUK, FOFI). Comportamento preesistente, non introdotto
dalla modifica, ma fa rumore nella risposta.

**Scoperta collaterale importante**: alla prima domanda francese l'agente **non ha chiamato il
flusso** (nessuna run) e ha risposto da solo, con la descrizione italiana e la traduzione fra
parentesi. E' esattamente il sintomo descritto da Chiara nel commento al **2418**: quel difetto
non e' nel flusso, e' nel routing: quando Search Items non viene invocato, nessuna traduzione
puo' applicarsi. Riformulando la domanda in modo piu' esplicitamente "di catalogo" il flusso
viene chiamato e la risposta esce tutta in francese.
