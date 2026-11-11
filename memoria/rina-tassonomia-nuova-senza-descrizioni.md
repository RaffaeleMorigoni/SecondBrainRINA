---
name: rina-tassonomia-nuova-senza-descrizioni
description: Le 591 categorie della nuova tassonomia RINA esistono ma senza descrizione, quindi la ricerca per keyword trova solo le categorie legacy
metadata:
  type: project
---

La gerarchia categorie `RINA` in F&O contiene 893 categorie, di due generazioni:
- **legacy** (2016-2021): `Np...`, `Pr...`, `CS...`, `In...` (~300). Hanno una descrizione italiana.
- **nuova tassonomia** (2025-2026): `SE` `SU` `PS` `MA` `ME` `EX` + cifre (591), allineata ai codici
  MMD e gerarchica: `SE` > `SE15` > `SE1520` > `SE152002` > materiale `SE152002.0001`.
  **`CategoryDescription` e `FriendlyCategoryName` sono vuoti.**

`IsActive` vale `Yes` per entrambe le generazioni, quindi **non e un criterio utilizzabile**:
l'unico modo per distinguerle e il pattern del nome.

**Why:** il flusso cerca le categorie per parola chiave sul nome descrittivo, quindi puo trovare
solo le legacy. Per questo l'agente apriva la risposta con `Prbpro220 - Manutenzione attrezzature
e macchinari`, una categoria del 2019, proprio mentre cercava di dare la tassonomia richiesta dal
business.

**How to apply:** nelle istruzioni del child agent la regola vieta ogni nome che inizi con
Np, Pr, CS, In e ammette solo `^(SE|SU|PS|MA|ME|EX)\d`; se restano solo categorie legacy l'agente
omette del tutto la riga della categoria. Finche RINA non popola le descrizioni delle 591 categorie
nuove, la richiesta "prima la tassonomia poi il codice" **non e soddisfacibile**: si puo mostrare
solo un codice categoria nudo tipo `SE1520`, che al business non dice nulla. Da filtrare anche
nell'output `categories` del flusso (non ancora fatto). Vedi [[rina-search-items-etichette-disponibilita]].

**Soluzione trovata 2026-09-29:** le descrizioni mancanti in F&O **esistono gia** nel file del
cliente `RINA_MD_Material_Codes_v1.xlsx` (in Downloads), foglio **`CATEGORY_TREE`**: 579 categorie
con descrizione in 6 lingue (IT, EN, PT, FR, SP, CH), livelli L1-L4 che corrispondono ai livelli
2-5 di F&O. Esempio: `SE` = Servizi, `SE15` = Servizi manutenzione macchinari e attrezzature,
`SE1520` = Manutenzione attrezzature ed accessori produzione e ufficio, `SE152002` = Manutenzione
attrezzature tecnologiche.

Copertura verificata: **573 delle 596 categorie nuove di F&O (96%)**. Le 23 scoperte
(`SE601003-004`, ramo `SE6510`/`SE6520`, `SU101002`) **non contengono alcun materiale**, quindi la
copertura pratica e del 100% su cio che una ricerca puo restituire. Il ramo `SE65` di F&O sembra
corrispondere al `PS65` dell'Excel (rinominato dopo luglio), e `SE601001` diverge come sinonimo
("immobili commerciali" in F&O, "immobili strumentali" nell'Excel): l'Excel e una fotografia di
luglio 2026, non la fonte viva.

Lo stesso file ha il foglio `MASTER_MATERIAL_CODES` con 682 materiali, descrizioni in 6 lingue e
una colonna di descrizione estesa con esempi.

**Tentativo del 2026-09-29 e rollback.** Il file `RINA_MD_Material_Codes_v1` e **gia** nella
knowledge dell'agente (caricato da Chiara), insieme a `Old_new_only_RINA`. La knowledge **funziona**:
alla domanda diretta "che descrizione hanno SE15 e SE1520" l'agente risponde in ~10s con le
descrizioni esatte del CATEGORY_TREE. Quindi la fonte dati e valida e non serve caricare nulla.

**Ma** aggiungere alle istruzioni la regola "apri con la categoria presa dal CATEGORY_TREE" ha
causato una regressione: la risposta perde la struttura in due sezioni (disponibili / da richiedere)
e torna a un paragrafo unico, e i tempi passano da ~42s a ~85s. Provato due volte, con regola lunga
e con regola compatta: stesso esito. Rollback alle istruzioni da 20162 caratteri, che danno la
risposta strutturata senza categoria.

**How to apply:** la strada e valida ma va ripresa con calma, probabilmente NON via knowledge search
dentro la stessa risposta (costa un giro di ricerca e destruttura l'output). Alternative da valutare:
mettere le ~590 righe categoria/descrizione come tabella compatta dentro le istruzioni, oppure
restituire la descrizione categoria direttamente dal flusso. File gia pronto per entrambe le strade:
`C:\Users\RaffaeleMorigoni\Downloads\RINA_Tassonomia_Categorie_HUDI.xlsx` (579 righe: livello,
codice, IT, EN, padre), generato dal CATEGORY_TREE e privo di codici materiale.
