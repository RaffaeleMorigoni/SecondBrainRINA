---
name: rina-rspa-anagrafica-mmd-vuota
description: In UAT la legal entity RSPA ha quasi solo vecchi codici Np, praticamente nessun nuovo MMD rilasciato
metadata:
  type: project
---

Verificato il 2026-09-29 su rina-uat-clm via OData (ReleasedProductsV2): la legal entity
**RSPA (RINA S.p.A.) ha 256 articoli rilasciati, di cui 243 vecchi codici `Np...` e un solo
nuovo MMD** (`MA101009.0001`). Confronto con le altre LE: RSSE 1100 articoli / 572 nuovi MMD,
RBST 151/121, RENT 281/42, APPO 346/2.

**Why:** i tester che lavorano su RSPA (es. Emanuela) vedono l'agente proporre codici che nella
loro societa non esistono. Non e un difetto dell'agente: l'anagrafica MMD non e stata estesa a
RSPA in UAT. Un filtro rigido per legal entity renderebbe l'agente muto su RSPA.

**How to apply:** prima di trattare come bug una segnalazione "questo codice non c'e nella mia LE",
controllare in quale societa sta testando chi segnala. Da confermare col cliente se in produzione
l'anagrafica sara estesa a tutte le LE: se RSPA resta cosi, e un problema di dati, non di agente.
Vedi [[rina-search-items-etichette-disponibilita]].

**Aggiornamento 2026-09-29:** confermato che **in produzione l'anagrafica MMD sara estesa a tutte
le legal entity**. Quindi lo squilibrio e limitato a UAT e non va segnalato al cliente come domanda
aperta: va tenuto presente solo per interpretare le segnalazioni dei tester che lavorano su RSPA.

**Attenzione, 15/10:** una parte delle risposte "non risultano articoli disponibili nella tua
societa'" NON era anagrafica vuota, ma il difetto del Cross Company su `Lista_articoli`
(vedi [[rina-company-sessione-dataareaid]]). Prima del fix, "manutenzione attrezzature" in RSSE
dava **zero** disponibili; dopo il fix ne da' **otto**. Prima di attribuire un "non trova
niente" all'anagrafica, controllare il filtro nella run del flusso.

