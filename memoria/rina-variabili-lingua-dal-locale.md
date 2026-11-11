---
name: rina-variabili-lingua-dal-locale
description: "Due variabili globali (CurrentUserLanguage e UserLanguage) dicono che la lingua si prende dal contesto F&O, cioe' dal locale: contraddicono il blocco LANGUAGE e la descrizione dell'input del flusso"
metadata:
  type: project
---

**Trovate il 30/10/2026** scaricando il file di localizzazione francese dell'orchestratore
(Settings > Languages > French > Manage > Download as JSON).

```
CurrentUserLanguage.Description
  "Lingua corrente dell'utente acquisita dal contesto di Dynamics 365 Finance & Operations.
   Utilizzata per passare al RINA Procurement & MMD Assistant la lingua da utilizzare nella risposta."

UserLanguage.Description
  "The current user's language in the Dynamics 365 Finance and Operations context.
   Use it to preserve language consistency when delegating requests to specialized agents."
```

**Entrambe dicono che la lingua viene dal contesto F&O, cioe' dal locale dell'interfaccia.**
La prima la chiama esplicitamente *"la lingua da utilizzare nella risposta"*.

**Contraddicono**: il blocco LANGUAGE (*"Never let system locale … override it"*), la descrizione
dell'input `UserLanguage` del flusso (*"the language the user actually wrote their own message
in"*) e la regola vera: **la lingua la da' la conversazione**.

**Quarto permesso-che-contraddice in un giorno**, dopo [[rina-permesso-dati-pagina]],
[[rina-lingua-campi-dynamics]] e [[rina-blocco-language-ambito]]. Lo schema si ripete: non manca
un divieto, c'e' un'istruzione che ordina il contrario.

**Spiega il residuo francese**: con interfaccia `en-US` la variabile vale inglese e viene passata
al figlio come lingua della risposta.

**Esclusa** invece l'ipotesi che la localizzazione sovrascriva le istruzioni: nel file francese
le istruzioni del padre sono **identiche** a quelle pubblicate (7.973 caratteri).

**Non toccate**: sono dell'orchestratore Microsoft in soluzione gestita. Prima guardarle in
*Variables* - chi le valorizza, chi le legge - poi decidere. Due variabili per la stessa cosa
sono gia' un segnale.

Dettaglio in `plans/20261030_due_variabili_lingua.md`.
