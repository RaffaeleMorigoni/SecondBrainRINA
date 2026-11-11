---
name: rina-utenze-tester-societa
description: I permessi F&O dei tester non incidono sulle risposte; incide la societa predefinita dell'utenza, e i tester chiave sono su RSPA che e vuota
metadata:
  type: project
---

**I permessi F&O dei tester non influenzano le risposte dell'assistente.** I quattro flussi di
lettura girano con la connessione dell'autore (`C_110912`), quindi leggono sempre con le credenziali
di Raffaele. Fa eccezione solo `Create RDA Draft`, che usa la connessione di chi lo invoca.

**Incide invece la societa predefinita dell'utenza** (`SystemUsers.Company`): `Search Items` la
legge e filtra gli articoli rilasciati su quella societa. Conta la **predefinita dell'utenza**, non
quella selezionata nell'interfaccia F&O: se l'utente cambia azienda dal menu, l'assistente continua
a rispondere sulla predefinita.

Verificato il 2026-09-29 in UAT:

| Societa | Utenti abilitati | Articoli | di cui nuovi MMD |
|---|---:|---:|---:|
| RSSE | 39 | 1100 | 572 |
| PATR | 31 | 615 | 607 |
| **RSPA** | **18** | **256** | **1** |
| DAT | 19 | 0 | 0 |
| PRID | 15 | 349 | 131 |
| RBST | 9 | 151 | 121 |
| ERUK | 4 | 249 | 0 |

**Utenze dei tester** (verificate il 29/09 incrociando `Workers` e `SystemUsers`):

| Persona | Utenza | Societa | Attiva | Lavoratore collegato |
|---|---|---|---|---|
| Emanuela Ferrari | `EF005` | **RSSE** | si | si |
| Roberto Mantese | `RM018` | **RSPA** | si | si |
| Andrea Maggiora | `AM015` | **RSPA** | si | si |
| Enrico Cogorno | `ECG` | **RSPA** | si | **NO** |
| Raffaele Morigoni | `C_110912` | **RSSE** | si | - |

Il lavoratore ENRICO COGORNO (matricola 1830) ha il campo `User` **vuoto**: l'utenza `ECG` esiste ed
e attiva, ma non e collegata al record del lavoratore. Michela Rocca (`MRA`, RSPA, attiva) ha invece
l'**email vuota**, quindi l'assistente non la riconosce affatto.

**Why:** i due tester che hanno prodotto le segnalazioni piu negative lavorano entrambi su RSPA, la
societa con l'anagrafica praticamente vuota. Le loro risposte "non trovato" erano in larga parte
corrette rispetto ai dati della loro societa, non difetti dell'assistente.

**How to apply:** prima di trattare una segnalazione come difetto, controllare la societa
predefinita di chi l'ha aperta. I prerequisiti perche l'assistente funzioni sono nel file
`RINA - Abilitazione buyer e tester UAT.xlsx` in Downloads (foglio Note): utenza attiva, email
uguale all'UPN di accesso, record persona con lavoratore collegato, societa predefinita corretta,
licenza Copilot. Vedi [[rina-rspa-anagrafica-mmd-vuota]] e [[rina-fo-user-worker-prerequisite]].

**Prova del 29/09 simulando RSPA.** Cambiata temporaneamente la societa predefinita di `C_110912`
da RSSE a RSPA via OData PATCH su `SystemUsers`, rieseguiti sei casi, poi **ripristinata a RSSE**.
Il PATCH funziona dalla sessione browser con `If-Match: *` e risponde 204.

Risultato: **i codici proposti sono gli stessi di RSSE e restano corretti**, ma finiscono quasi
tutti nella sezione "da richiedere", il che e il comportamento giusto. La matrice dei conti
funziona anche su RSPA (MA101008.0002 da conto 410000 come su RSSE).

**Difetto che si vede solo su RSPA:** quando la sezione "disponibili nella tua societa" e vuota, in
circa meta dei casi la risposta **perde l'etichetta MMD |** e si destruttura. E il difetto del
doppio messaggio (punto aperto 28): parla l'orchestratore, che l'etichetta non la mette. Su RSSE
non si notava perche le risposte sono piene. Spiega perche i tester su RSPA lo incontrano spesso.

Osservato anche un forte rallentamento su alcune risposte (fino a circa otto minuti), da tenere
d'occhio ma non necessariamente legato alla societa.

**Difetto dell'etichetta risolto il 29/09 senza toccare l'architettura.** Le risposte corrette
mantenevano la struttura a sezioni anche a zero risultati, quelle rotte diventavano un paragrafo
discorsivo: l'orchestratore interviene quando percepisce la risposta del figlio come **incompleta**.
Aggiunta al figlio la regola che impone di scrivere comunque l'intestazione della sezione degli
articoli disponibili con una riga che dichiara l'assenza, mantenendo la stessa forma con sezioni
piene o vuote. Verificato su RSPA sui due casi che si rompevano: entrambi tornano corretti.
Istruzioni a 21034 caratteri. Non e servito il passaggio a "Send specific response".

**Verificato il 12/10 sulla run del flusso:** anche `C_110912` (l'utenza con cui proviamo) ha
`SystemUsers.Company = RSPA`. Quindi le nostre prove in F&O cercano in RSPA, che e' quasi vuota,
**anche quando la sessione a video e' su RSSE**. E' esattamente il difetto che
[[rina-company-sessione-dataareaid]] deve correggere: la ricerca segue la societa' predefinita,
non quella selezionata.

