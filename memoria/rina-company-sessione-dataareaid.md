---
name: rina-company-sessione-dataareaid
description: La company della sessione F&O arriva all'agente in Global.PA_Copilot_ServerForm_UserContext.dataAreaId, non va dedotta dalle opzioni utente
metadata:
  type: reference
---

Copilot in F&O passa al chatbot **"Copilot in Finance and Operation"** tre variabili
globali di contesto, popolate in automatico per la sessione dell'utente
([doc Microsoft](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/copilot/copilot-application-context)):

- `Global.PA_Copilot_ServerForm_UserContext` (Record)
  - **`dataAreaId`** — "the legal entity that the user is **currently signed in to**":
    la company della sessione a video, non quella predefinita nelle opzioni utente;
  - `userLanguageId` — la lingua scelta nelle User options (utile per il task 2325:
    \u00e8 generica e pi\u00f9 affidabile dell'euristica sul testo della domanda).
- `Global.PA_Copilot_ServerForm_PageContext` (Record) — pagina aperta e record a video:
  `caption`, `metadataName`, `rootTableName`, `rootTableRecId`, `titleFieldNValue`.
- `Global.PA_Copilot_ServerForm_NavigationContext` (Table) — voci di menu visibili
  all'utente, max 500, filtrate dai suoi permessi.

**Perch\u00e9 conta per RINA:** oggi la ricerca articoli \u00e8 legata alla societ\u00e0 predefinita
dell'utente. Gli utenti con deleghe interne creano RDA in altre company, e per loro il
risultato \u00e8 sbagliato. Con `dataAreaId` la ricerca segue la company che l'utente ha
davvero davanti.

**Tre vincoli da ricordare:**
1. Le variabili sono popolate **solo quando l'agente gira dentro F&O**. Nel pannello di
   test di Copilot Studio sono vuote: serve un fallback alla company predefinita, altrimenti
   i test si rompono. Stessa famiglia di problemi di [[rina-formato-fo-non-governabile]].
2. Sono globali **del padre**: vanno passate esplicitamente come input allo strumento o
   all'agente figlio, non si ereditano da sole. Vedi [[rina-descrizioni-strumenti-limiti]].
3. Se la sessione \u00e8 su RSPA la ricerca torner\u00e0 quasi vuota, ma correttamente: vedi
   [[rina-rspa-anagrafica-mmd-vuota]].

**Come passarlo, deciso il 12/10:** in Copilot Studio si usano parametri espliciti dello
strumento, non variabili globali ereditate. `dataAreaId` va dichiarato come input dello
strumento e valorizzato dal padre, con fallback alla società predefinita quando è vuoto.

## Provato il 12/10: `Activity.ChannelData` NON funziona

Tentativo: nello strumento del figlio, `CompanyId` = formula
`Text(System.Activity.ChannelData.dataAreaId)`. **Compila** (ChannelData e' di tipo any,
quindi l'accesso ai campi passa la validazione) ma **a runtime torna vuoto**.

Prova in F&O con sessione su **RSSE**: la run del flusso mostra
`$filter: dataAreaId eq 'rspa'`, cioe' il fallback alla societa' predefinita. Non ritentare.

**Perche' la catena non si chiude:** gli input che l'orchestratore dichiara sull'agente figlio
(CurrentUserRequest, UserLanguage, CompanyId) **non diventano variabili globali del figlio** -
nel figlio il pannello Variables dice Global (0) e la tab Custom del selettore e' vuota. Uno
strumento del figlio non puo' leggerli.

**Strada che resta:** spostare lo strumento di ricerca **sull'orchestratore**, dove
`Global.PA_Copilot_ServerForm_UserContext.dataAreaId` e' selezionabile direttamente (verificato
nel selettore). Da concordare con RINA perche' l'orchestratore e' in managed solution.

## RISOLTO il 12/10: lo strumento di ricerca spostato sull'orchestratore

Gli strumenti si creano **sull'orchestratore** e si assegnano con **"Available to"**: la lista
Tools del padre mostra tutti quelli dei figli. Aggiungendo il flusso `RINA - Search Items` come
strumento del padre, "Available to" diventa *Copilot for finance and operations apps* e la
variabile di contesto e' selezionabile.

Configurazione dello strumento sul padre:
- `Keywords` -> Dynamically fill with AI
- `UserEmail` -> Custom value -> `User.PrincipalName` (vedi [[rina-upn-non-email]])
- `CompanyId` -> Custom value -> `Global.PA_Copilot_ServerForm_UserContext.dataAreaId`
- Descrizione copiata da quella del figlio (708/1024), con "legal entity of the F&O session the
  user is currently signed in to" al posto di "of the user who is chatting".

**Prova del 12/10, sessione F&O su RSSE con utente la cui societa' predefinita e' RSPA:**
- prima: `$filter: dataAreaId eq 'rspa'` (sbagliato)
- dopo: `$filter: dataAreaId eq 'rsse'` (corretto), e la risposta nomina RSSE.
Screenshot in `plans/prove_uat/20261012_company_*`.

**Il costo, da tenere d'occhio:** la ricerca ora la esegue il padre, quindi la risposta la
formatta lui. Nel test il template a due sezioni del figlio non e' stato applicato: e' uscito
l'elenco grezzo `codice | descrizione | tipo`. La label `MMD |` e la distinzione
disponibili/da richiedere restano. Lo strumento esiste ora sia sul padre sia sul figlio: da
decidere se disattivare quello del figlio. Vedi [[rina-template-risposta-ricerca]].

## 15/10: TRADE-OFF accertato fra societa' e lingua

Chi esegue la ricerca determina quale dei due funziona:

| Ricerca eseguita da | Societa' della sessione | Lingua dell'utente |
|---|---|---|
| **Orchestratore** (strumento con `CompanyId` = dataAreaId) | **OK** (RSSE) | **KO** (risponde italiano) |
| **Agente figlio** | KO (ricade su RSPA) | **OK** (risponde portoghese) |

Verificato alternando lo stato dei due strumenti e provando la stessa domanda in portoghese.
Il Response formatting in inglese aiuta il **figlio**, non il padre: con la ricerca sul padre la
lingua torna italiana comunque.

**Tentativo fatto e fallito:** aggiungere `CompanyId` allo strumento del figlio con "Dynamically
fill with AI", sperando che l'agente lo valorizzasse col valore ricevuto dal padre. Il flusso ha
ricevuto vuoto e ha usato il fallback (rspa).

**Attenzione operativa:** i due strumenti compaiono **entrambi** nella lista Tools
dell'orchestratore (uno per "Available to"). Se restano accesi tutti e due, l'orchestratore
sceglie quello del figlio e si perde la societa'. Verificare sempre la lista del padre, non
quella del figlio: spegnere dal figlio non basta, torna On dopo la pubblicazione.

**Stato lasciato il 15/10:** ricerca sull'**orchestratore** (societa' corretta, lingua italiana).

