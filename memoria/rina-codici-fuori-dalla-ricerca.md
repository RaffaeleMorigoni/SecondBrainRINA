---
name: rina-codici-fuori-dalla-ricerca
description: "Quando la ricerca torna vuota l'agente risponde con codici che non vengono da F&O: provato confrontando la risposta con gli output delle Lista prodotti - zero sovrapposizione"
metadata:
  type: project
---

**30/10/2026.** Run riuscita delle 16:36, domanda francese sulla manutenzione attrezzature.

| | |
|---|---|
| codici **trovati dalla ricerca** | `SE202001.0001`, `SE202002.0001`, `SE253002.0001`, `PS103002.0001`, `PS104001.0001` |
| codici **mostrati all'utente** | `SE105001.0001`, `SE152002.0001`, `SE153002.0001`, `SE153003.0001`, `SE153005.0001`, `SE303002.0001` |

**Zero sovrapposizione.** Due delle tre `Lista prodotti` avevano restituito `"value": []`;
la terza cinque codici, nessuno dei quali e' finito nella risposta.

**Quindi le risposte "riuscite" erano il difetto peggiore**, e le "fallite" (*"Aucun code MMD…"*)
erano il comportamento corretto. E' [[rina-knowledge-compete-con-la-ricerca]], ancora aperto:
quando la ricerca torna vuota l'agente ripiega su Knowledge o memoria del modello invece di
fermarsi.

**Perche' e' pericoloso**: il formato e' impeccabile - prefisso `MMD | `, elenco pulito,
descrizioni nella lingua giusta. Niente nella risposta segnala che i codici non sono stati
cercati. Io stesso l'ho giudicata buona due volte.

**Regola operativa**: una risposta e' corretta solo dopo aver **confrontato i codici mostrati con
gli output delle `Lista prodotti`** della run corrispondente in Power Automate. Guardare il
pannello non basta, e tutti i giudizi dati solo a video vanno riaperti - compresa la controprova
su `PS101002.0002` ([[rina-fallback-generico-orchestratore]]).

**Gerarchia**: questo difetto viene **prima** del 2494 ([[rina-ricerca-parole-italiano-iniziaper]]).
Finche' e' aperto, nessun test basato sulla risposta a video e' attendibile.

Dettaglio in `plans/20261030_2494_le_query.md`.
