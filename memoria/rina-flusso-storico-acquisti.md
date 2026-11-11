---
name: rina-flusso-storico-acquisti
description: Stato di costruzione del flusso RINA - Item Purchase History per il capitolo 2 del BRD
metadata:
  type: project
---

Flusso **`RINA - Item Purchase History`**, id `25dfc123-bac1-f111-aaaf-7ced8d12eeda`, creato il
06/10/2026 con Save As da `RINA - Get Item Accounting Types`. **In bozza, mai pubblicato, non
collegato all'agente.** Serve il capitolo 2 del BRD: ultimi ordini, fornitore principale, conto
piu usato.

**Fatto e salvato in bozza (06/10 sera)**
- trigger con due input obbligatori: `ItemNumber` (text) e `UserEmail` (text_1)
- `Lista_categoria` riusata come lettura utente: entita **PersonUsers**, filtro su `UserEmail`
- `Lista_matrice` riusata come lettura societa: entita **SystemUsers**, filtro su `UserID` preso da
  `Lista_categoria`, restituisce `Company`
- `Lista_tipi` riusata come lettura ordini: entita **PurchaseOrderLinesV2**, filtro
  `dataAreaId eq <company in minuscolo> and ItemNumber eq <item>`, Top Count 100,
  Order By `PurchaseOrderNumber desc`, Select Query con i campi confermati dalla documentazione CDM:
  `PurchaseOrderNumber,ItemNumber,OrderedPurchaseQuantity,PurchaseUnitSymbol,PurchasePrice,LineAmount,MainAccountId,MainAccountIdDisplayValue,PurchaseOrderLineStatus,RequestedShippingDate,ConfirmedShippingDate`
- eliminate le tre azioni residue `Filtra_tipi`, `Seleziona_tipi`, `Seleziona_id`
- `Rispondi_all_agente` riscritto con tre output: `OrderLineCount`
  (`@length(coalesce(outputs('Lista_tipi')?['body/value'], json('[]')))`), `OrderLines`
  (`@string(...)`, **dump grezzo provvisorio, da trasformare in righe leggibili dopo il primo test**)
  e `OrderSummary` (`@string('')`, segnaposto)

**Pubblicato e testato il 06/10 sera** (su autorizzazione esplicita dell'utente: il test di un
agent flow passa **solo** da Publish & Test, non esiste esecuzione senza pubblicare). Le tre
letture funzionano, statusCode 200.

**Campi di PurchaseOrderLinesV2 verificati sul campo** (la chiamata passa con questi tredici):
`PurchaseOrderNumber, LineNumber, ItemNumber, ProcurementProductCategoryName,
OrderedPurchaseQuantity, PurchaseUnitSymbol, PurchasePrice, LineAmount,
MainAccountIdDisplayValue, PurchaseOrderLineStatus, RequestedShippingDate,
ConfirmedShippingDate, LineDescription`.
**`MainAccountId` NON esiste** sull'entita esposta dal connettore (esiste nel modello CDM ma non
qui): usare solo `MainAccountIdDisplayValue`.

**Tre fatti emersi dai dati veri di UAT**
- la societa di `C_110912@rina.org` e **rsse**, non RSPA
- `MainAccountIdDisplayValue` **arriva vuoto** sulle righe di articolo: il conto e sulla
  distribuzione contabile, non sulla riga. Il "conto piu usato" del BRD non e ricavabile da qui
  per gli articoli; per i servizi va riverificato
- le date nulle di F&O arrivano come **`1900-01-01T12:00:00Z`**: vanno scartate, non mostrate
- articolo con storico utile per i test: **`Npassi111`**; `SE201001.0001` non ha ordini in rsse

**Lettura testate aggiunta e provata il 06/10 (22:15)** - il fornitore non sta sulla riga:
- azione **`Componi ordini`** (Select, Map in modalita testo):
  From `@coalesce(outputs('Lista_tipi')?['body/value'], json('[]'))`,
  Map `@concat('PurchaseOrderNumber eq ', decodeUriComponent('%27'), item()?['PurchaseOrderNumber'], decodeUriComponent('%27'))`
- azione **`Lista testate`** su **PurchaseOrderHeadersV2**, filtro societa + clausola OR costruita
  con `join(union(body('Componi_ordini'), body('Componi_ordini')), ' or ')` (il doppio union
  deduplica), sentinella `ZZNONEZZ` sul ramo vuoto, Top 100,
  Select `PurchaseOrderNumber,OrderVendorAccountNumber,PurchaseOrderStatus,AccountingDate`
- terzo output rinominato **`OrderHeaders`** = `@string(coalesce(outputs('Lista_testate')?['body/value'], json('[]')))`
Esecuzione riuscita su `Npassi111`: ordini veri (RSSEPO19021321), stato `Invoiced`,
`AccountingDate` 2019-06-12 (data vera, usabile per "ultimo anno").

**Lettura nomi fornitore aggiunta e provata il 06/10 (22:30)** - le testate danno solo il codice
conto (`SUP000033853`), non la ragione sociale:
- azione **`Componi fornitori`** (Select, Map in modalita testo):
  From `@coalesce(outputs('Lista_testate')?['body/value'], json('[]'))`,
  Map `@concat('VendorAccountNumber eq ', decodeUriComponent('%27'), item()?['OrderVendorAccountNumber'], decodeUriComponent('%27'))`
- azione **`Lista fornitori`** su **VendorsV2**, stesso schema di filtro (societa + OR dedup con
  doppio `union`, sentinella `ZZNONEZZ`), Top 100,
  Select `VendorAccountNumber,VendorOrganizationName,VendorSearchName`
- quarto output **`Vendors`** = `@string(coalesce(outputs('Lista_fornitori')?['body/value'], json('[]')))`
Esecuzione riuscita: `SUP000024165` -> **RCS MEDIAGROUP SPA**.

Il flusso completo e quindi: Lista categoria -> Lista matrice -> Lista tipi -> Componi ordini ->
Lista testate -> Componi fornitori -> Lista fornitori -> Rispondi all agente (4 output).

**Nomi e descrizione sistemati il 06/10 (22:37)**
Scope `Leggi storico acquisti`; azioni `Lista utente` (PersonUsers), `Lista societa`
(SystemUsers), `Lista righe ordine` (PurchaseOrderLinesV2), `Componi ordini`, `Lista testate`,
`Componi fornitori`, `Lista fornitori`, `Rispondi all agente`.
Descrizione del flusso riscritta: dice che restituisce lo storico acquisti di un articolo nella
societa dell'utente - **conta davvero**, perche e cio che l'agente legge per decidere quando usare
lo strumento.

**Rinominare un'azione e sicuro:** Power Automate aggiorna da solo i riferimenti nelle espressioni
e nei `runAfter` (verificato in Code view: `outputs('Lista_categoria')` e diventato
`outputs('Lista_utente')`). Test dopo le rinomine: riuscito.

**Output in testo leggibile, fatto e provato il 06/10 (22:50)**
Tre Select in coda (`Testo righe`, `Testo testate`, `Testo fornitori`), Map in modalita testo, e i
tre output diventati `@join(body('Testo_...'), decodeUriComponent('%0A'))`. Niente piu
`@odata.etag` ne `ItemInternalId`. Formato prodotto:
- righe: `ORDINE | descrizione | qta N UM | prezzo N | importo N | stato X | categoria Y`
- testate: `ORDINE | fornitore SUPxxx | stato X | data AAAA-MM-GG`
- fornitori: `SUPxxx | RAGIONE SOCIALE`

**Scelta di progetto:** i tre blocchi restano separati e **l'agente li ricollega** per numero
d'ordine e codice fornitore. Non si fa il join nel flusso perche dentro un `Select` un `filter()`
annidato rimappa `item()`: il lookup diventerebbe fragile.

**Le date nulle sono gestite:** `if(startsWith(..., '1900'), 'n.d.', first(split(..., 'T')))`.

**Collegato all'agente il 06/10 (23:10)**
Agente `RINA Procurement & MMD Assistant`, bot id `dfd051a3-a19f-f111-b8dc-7ced8d12eeda`,
ambiente `a485debc-...` (rina-uat-clm). Tool aggiunto, salvato e **agente pubblicato il 06/10 alle 23:12** (senza *Force newest version*, per non interrompere le conversazioni in corso).
- `ItemNumber`: Dynamically fill with AI
- `UserEmail`: **Custom value = `System.User.Email`**, non lasciato all'AI (altrimenti potrebbe
  inventarla o chiederla all'utente)
- la **Description dello strumento e stata riscritta a mano**: Copilot Studio NON eredita la
  descrizione del flusso, ci mette il nome del flusso (28 caratteri). E la descrizione che
  l'orchestratore legge per decidere se usarlo: senza, lo strumento non verrebbe mai invocato.

**Sorpresa da verificare con l'utente:** l'agente in questo ambiente aveva **un solo tool**
(`MCP Dynamics 365 ERP`, di Chiara DAMBROSIO, un mese fa) e **nessuno dei flussi** su cui abbiamo
lavorato (Search Items, Get Item Accounting Types), pur avendo le nostre istruzioni (blocco SCOPE
identico). Non c'e nessun altro agente RINA/MMD in questo ambiente ne nell'altro del tenant.
Se i flussi sono stati rimossi, le run della suite di regressione dei giorni scorsi giravano su
una configurazione diversa da quella attuale.

**Nota sulla pubblicazione:** Copilot Studio avvisa che lo strumento gira con le **credenziali
dell'autore**. Non e un problema di isolamento dati: il filtro per societa non dipende dalla
connessione ma dall'`UserEmail` passato come input, che e `System.User.Email`.

**Provato end to end il 06/10 alle 23:16 - DIFETTO TROVATO**
Domanda in chat: "Come abbiamo acquistato in passato l'articolo Npassi111? Da quale fornitore?".
L'agente **ha invocato il flusso** (run `Succeeded`, non di test) ma ha risposto *"non risultano
acquisti precedenti"*, che e **falso**: quell'articolo ha decine di ordini.

**Causa:** `System.User.Email` restituisce l'email **Entra** di chi chatta, che per un consulente
esterno e `raffaele.morigoni@ext.rina.org`. In F&O quella utenza **non esiste** in `PersonUsers`
(l'utenza F&O e `C_110912@rina.org`). Catena rotta al primo passo: nessun PersonUser -> nessuna
Company -> filtro `dataAreaId eq ''` -> zero righe.
Per i tester interni con email @rina.org potrebbe funzionare, **ma va verificato**: dipende se la
loro email Entra coincide con `PersonUsers.UserEmail`.

**Secondo difetto - CORRETTO la sera del 06/10 (23:30).** Il flusso non distingueva "utente non
riconosciuto" da "nessun ordine", e il vuoto diventava "questo articolo non e mai stato
acquistato": falso, del tipo che i tester segnalano come allucinazione.
Correzione: quinto output **`Status`** su `Rispondi all agente`
`@if(empty(trim(coalesce(string(first(outputs('Lista_societa')?['body/value'])?['Company']), ''))), 'USER_NOT_RESOLVED - ...', 'OK')`
dove il ramo di errore **contiene gia l'istruzione in chiaro** per l'agente (non dire che
l'articolo non e mai stato comprato, di' che manca la mappatura utente F&O). Alla Description
dello strumento in Copilot Studio e stata aggiunta la frase *"Always read the Status output
first..."*. Flusso e agente ripubblicati.
**Verificato in chat:** stessa domanda, l'agente ora risponde che non puo verificare perche
l'utenza non e associata a un utente F&O e dice esplicitamente che **questo non significa che
l'articolo non sia mai stato acquistato**. Comportamento corretto.

**Tecnica riutilizzabile:** mettere l'istruzione per l'agente *dentro il valore* dell'output di
stato, invece che nelle istruzioni dell'agente. Evita di toccare le istruzioni - che su questo
progetto hanno sempre prodotto effetti collaterali misurabili sulla suite.
- valutare se aggiungere una clausola d'uso nelle istruzioni dell'agente: non fatto apposta,
  perche ogni aggiunta alle istruzioni ha avuto effetti collaterali misurabili sulla suite

**Why:** costruito la sera del 06/10 su richiesta esplicita, dopo che la decisione di non aggiungere
integrazioni e stata riaperta.
**How to apply:** riprendere da qui. Vale la regola di [[power-automate-espressioni-non-si-salvano]]:
se una modifica a un campo viene scartata in silenzio, **ricaricare la pagina del designer** e
rifarla - su sessione fresca passa. Per i campi dell'interfaccia conviene dare il focus via DOM
(`element.focus()` + Range sul contenteditable) invece che per coordinate: la barra di salvataggio
sposta il canvas e i click a coordinate finiscono sul campo sbagliato.


## Riaggancio dello strumento, 07/10 mattina

**Perche:** dopo aver ripubblicato il flusso piu volte (output `Status`, filtro diagnostico e
ripristino), la configurazione degli **input dello strumento si e svuotata**: `ItemNumber` e
`UserEmail` non comparivano piu, e con loro il legame a `User.Email`. Verificato a pagina
correttamente renderizzata, non era un problema di rendering.

**Fatto:** strumento eliminato e riaggiunto (nuovo id `bdd81c34-04ff-4930-a716-f9e13c631f6a`),
input tornati, `UserEmail` = `User.Email`, descrizione riscritta, agente ripubblicato
(07/10 ore 08:38).

**Regola:** ogni volta che si ripubblica il flusso conviene **riaprire lo strumento in Copilot
Studio e controllare che gli input ci siano ancora**. E Copilot Studio **non eredita mai la
descrizione del flusso**: ci mette il nome (28 caratteri) e va riscritta a mano ogni volta che si
riaggancia - senza, l'orchestratore non sa quando usarlo.

**Quirk di rendering:** Copilot Studio **non disegna nulla se la sua scheda non e quella attiva**
(`document.visibilityState === 'hidden'`): pagina nera, `main` vuoto, niente testo. Non basta la
finestra in primo piano, serve che sia la scheda selezionata del gruppo. Con due schede nel gruppo
MCP, chiuderne una risolve. Collegato a [[copilot-studio-browser-quirks]].

**Mappatura utenza esterna: NON fatta.** La formula
`If(Lower(System.User.Email) = "<utenza esterna>", "<utenza F&O>", System.User.Email)`
nell'input `UserEmail` e stata **bloccata dal classificatore di sicurezza**. A ragione: fa
interrogare F&O con l'identita di un'altra utenza. Per far funzionare lo strumento a un consulente
esterno la strada corretta e che **RINA mappi l'utenza in F&O** (`PersonUsers.UserEmail`), non un
alias lato agente.


## Prova end to end del 07/10 (08:44): FlowActionTimedOut

Lo strumento riagganciato **viene invocato correttamente**: nel pannello di test si vedono tutti e
cinque gli output (`OrderLineCount`, `OrderLines`, `OrderHeaders`, `Vendors`, `Status`) e la
descrizione completa nell'execution summary. Quindi schema e descrizione sono a posto.

**Ma l'agente va in errore `FlowActionTimedOut`** e la run resta `Running` per minuti.

**Collo di bottiglia misurato:** la prima lettura `Lista utente` (PersonUsers filtrata per
`UserEmail`) da sola prende **18-25 secondi**; le run complete stavano fra 20 e 53 secondi, ora
oltre. Il flusso fa 5 letture F&O in catena piu 3 Select.

**Da valutare per accorciarlo:**
- abbassare `Top Count` sulle righe da 100 a ~20: meno ordini -> clausola OR piu corta su testate
  e fornitori, che e quadratico sul tempo
- capire perche PersonUsers e cosi lenta (indice? entita pesante?) ed eventualmente sostituirla
- valutare se testate e fornitori servono sempre o solo quando l'utente chiede il fornitore


## Timeout risolto il 07/10 (09:00)

`Top Count` sulle righe d'ordine portato da **100 a 20** (testate e fornitori lasciati a 100: sono
gia limitati dalla clausola OR, che ora ha al massimo 20 ordini). Flusso ripubblicato.

**Risultato:** niente piu `FlowActionTimedOut`. Lo strumento **completa in 67,78 secondi**, stato
`Completed`, e l'agente risponde. Il percorso end to end funziona: `Status` arriva valorizzato con
`USER_NOT_RESOLVED` e l'agente lo riporta correttamente, dicendo che non puo leggere lo storico
perche l'utenza non e associata a un utente F&O e che **non si puo concludere che l'articolo non
sia mai stato acquistato**.

**Resta lento:** 67 secondi sono molti per una risposta in chat, anche se non vanno piu in
timeout. Se serve accorciare ancora, il candidato e sempre `Lista utente` su PersonUsers
(18-25 secondi da sola).

**Il percorso con dati veri non e ancora stato visto in chat**, perche l'utenza esterna non e
mappata in F&O: finora in chat si e visto solo il ramo `USER_NOT_RESOLVED`. Il ramo `OK` e
provato solo da Power Automate con `C_110912@rina.org`.

**07/10/2026 - in UAT lo storico sta sui CODICI VECCHI, non sui nuovi MMD.** Dimostrazione
verificata: `MA101021.0001` restituisce **zero** righe d'ordine (Status OK, utenza riconosciuta:
e' l'articolo a non avere movimenti), mentre `Npassi111` restituisce **20 righe d'ordine, 18
ordini distinti, 4 fornitori** con ragione sociale, prezzi da 1 a 100 e il dettaglio per ordine
con data, quantita, prezzo, importo e stato.

La ragione e' che gli ordini d'acquisto in UAT sono stati caricati prima della migrazione alla
nuova nomenclatura. **Conseguenza pratica: per dimostrare la funzione serve un codice vecchio.**
Se a una presentazione si chiede lo storico di un MMD nuovo si ottiene "nessun acquisto", che e'
la risposta giusta ma sembra una funzione che non funziona. Trovare un codice con movimenti:
run history del flusso `RINA - Item Purchase History`, guardare il `$filter` di `Lista righe
ordine` nelle esecuzioni andate a buon fine.

**Due cose da sapere prima di mostrarlo:** la risposta **ripete il codice vecchio** ("Per
l'articolo Npassi111 risultano..."), contro la regola che vieta di esporre identificatori della
vecchia nomenclatura - qui lo ripete perche' l'ha fornito l'utente, ma va deciso se e' accettabile.
E lo storico comprende **ordini di test** (`TestImpPO3`, `test11111`, `test222222`): l'agente lo
dichiara spontaneamente in fondo alla risposta, il che e' corretto ma va spiegato a chi guarda.
