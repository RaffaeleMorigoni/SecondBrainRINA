# Perche' i tester RINA non riescono a usare l'agente

Tre blocchi diversi trovati nel monitoring del 28/10, e la mail di Emanuela li conferma tutti
con nomi e date. **Nessuno dei tre riguarda la qualita' delle risposte.**

---

## 1. "You don't have access to talk to this bot" — manca la condivisione

L'agente ha una lista di **50 utenti condivisi** (tutti come *Viewer*).

Confronto con i nomi della catena mail:

| persona | nella condivisione |
|---|---|
| Alexandru Cristian NISTOR | **si'** |
| Lena LENSHA | **si'** |
| Maria POULAKI | **si'** |
| Roberto Giusto MANTESE | **si'** |
| Emanuela FERRARI, Enrico COGORNO, Andrea MAGGIORA | **si'** |
| **João Vitor DOS SANTOS BORDALLO DUTRA** | **NO** |
| **Anamaria MANAILA** | **NO** |

**Questo spiega la mail di João.** Lui scrive *"Although access has already been granted, when I
click on Copilot in the portal, I get a connection error"*: gli hanno dato accesso **all'ambiente
F&O**, ma **non all'agente** in Copilot Studio. Sono due autorizzazioni distinte, e averne una non
implica l'altra.

Anamaria Manaila era stata chiesta da Enrico il 26/10 e non risultava aggiunta.

**Risolto il 28/10 alle 17:22.** Aggiunti entrambi come *Viewer* (End user access), su indicazione di
Raffaele: la lista passa da **50 a 52** e il pannello conferma *"Successfully changed the sharing
settings for your agent"*.

- Joao Vitor DOS SANTOS BORDALLO DUTRA — `joaovitor.dossantos@rina.org`
- Anamaria MANAILA — `Anamaria.Manaila@rina.org`

Attenzione all'omonimia: nella directory RINA ci sono altri due "Joao Vitor" (DE OLIVEIRA GOMES,
FERNANDES DE BRITO) e un'altra "Ana Maria" (GASTELUT VAILLO). Il cognome BORDALLO non e' ricercabile:
il picker trova la persona solo digitando `joao vitor`.

Prove: `prove_uat/20261028_condivisione_50_utenti.jpg` (prima),
`prove_uat/20261028_condivisione_manca_joao.png`,
`prove_uat/20261028_condivisione_manca_anamaria.png`,
`prove_uat/20261028_condivisione_52_utenti_aggiunti.jpg` (dopo)

---

## 2. "Open connection manager to verify your credentials" — e' il server MCP

Nel pannello di condivisione c'e' l'avviso *"Your agent runs tools that use the author's
credentials"*, con l'elenco dei **6 strumenti** coperti:

- RINA - Check Vendor
- RINA - Get Item Accounting Types (×2)
- RINA - Search Items (×2)
- RINA - Category Ledger Accounts

**Sono tutti e soli i flussi Power Automate. I due strumenti MCP non ci sono.**

Quindi l'MCP **non** eredita le credenziali dell'autore: pretende la connessione del singolo
utente. E quella connessione risulta **`Not connected`** (verificato il 21/10). Quando
l'orchestratore sceglie l'MCP invece di un flusso, l'utente riceve la card *Open connection
manager* e si ferma.

E' esattamente quello che e' successo il **28/10 alle 8:06**: domanda *"mmd what is the
product ?"*, card ripetuta tre volte, nessuna risposta. **La sessione risulta "Resolved".**

**Correzione del mio giudizio del 21/10:** avevo scritto che l'MCP era innocuo in esercizio perche'
non connesso, e che il danno era solo nel pannello di test. **Sbagliato**: non connesso non
significa ignorato. Viene scelto lo stesso e **blocca l'utente**.

Prova: `prove_uat/20261028_tool_credenziali_autore.jpg`

---

## 3. FlowActionTimedOut — segnalazione 2494 aperta oggi

Sessione del 24/10 11:41, due domande normali (*"devo acquistare una consulenza informatica"*,
*"devo acquistare un monitor"*), due timeout, utente che abbandona.
Dettaglio in `20261028_sessioni_utenti_RINA.md`.

---

## Cosa farei, in quest'ordine

1. ~~**Aggiungere João e Anamaria** alla condivisione dell'agente.~~ **Fatto il 28/10 alle 17:22.**
2. **Disattivare i due strumenti MCP** (o connetterli). Finche' restano attivi e non connessi,
   qualunque utente puo' sbatterci contro senza capire perche'. Sono in sola lettura e non
   servono a nessuna funzione: li avevamo messi noi.
3. Il timeout, sul 2494.

---

## Sulla domanda di Emanuela riguardo alle lingue

*"in test ci sono tutte le lingue caricate sugli mmd?"*

**Si'.** Verificato il 16/10: l'entita' `ProductTranslations` ha le descrizioni in **7 lingue**
(italiano, inglese, francese, portoghese, spagnolo, cinese…), con **copertura piena sui codici
MMD nuovi** — 15 su 15 verificati avevano francese, portoghese e cinese.
**L'unica lacuna nota e' lo spagnolo** (1.288 descrizioni vuote su 1.897), gia' aperta come
impediment **2416**.

**Quindi il caso di Nistor non e' un problema di dati.** Il difetto e' il **2418**: quando
l'agente risponde **senza interrogare il catalogo**, le descrizioni non arrivano da
`ProductTranslations` ma da un'altra fonte, in inglese o in italiano. Verificato a freddo il
22/10: nessuna run del flusso, descrizione inglese con la traduzione fra parentesi.

E' lo stesso meccanismo del punto 2 di questo documento: **l'agente che non usa lo strumento
giusto**.
