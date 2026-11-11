# Due variabili globali dicono che la lingua si prende dal locale

30/10/2026 sera. Scaricato il file di localizzazione **francese** dell'orchestratore
(Settings > Languages > French (France) > Manage > Download as JSON) per capire perche' il
francese risponde in inglese. Il file e' in `Downloads/localizations.json`.

## Primo esito: la localizzazione NON sovrascrive le istruzioni

La chiave `gptComponent(…gpt.default).Metadata.Instructions` nel file francese e' **identica** a
quella pubblicata: 7.973 caratteri, frase nuova compresa. **Nessun override.** L'ipotesi di una
copia stantia delle istruzioni per il francese e' esclusa.

## Secondo esito, molto piu' importante: due variabili globali sulla lingua

```
globalVariable(…CurrentUserLanguage).Description
  "Lingua corrente dell'utente acquisita dal contesto di Dynamics 365 Finance & Operations.
   Utilizzata per passare al RINA Procurement & MMD Assistant la lingua da utilizzare
   nella risposta."

globalVariable(…UserLanguage).Description
  "The current user's language in the Dynamics 365 Finance and Operations context.
   Use it to preserve language consistency when delegating requests to specialized agents."
```

**Entrambe dicono che la lingua si prende dal contesto di F&O**, cioe' **dal locale
dell'interfaccia**. E la prima dice esplicitamente che e' *"la lingua da utilizzare nella
risposta"*.

### Questo contraddice tre cose che abbiamo scritto noi

1. il blocco LANGUAGE del padre: *"Never let system locale … override it"*;
2. la descrizione dell'input `UserLanguage` del flusso: *"Code of the language the user
   **actually wrote their own message in** … Read it from the user's own words"*;
3. la regola che l'utente ha enunciato stasera: **la lingua la da' la conversazione**.

**E' il quarto permesso-che-contraddice trovato oggi**, dopo il 2418
([[rina-permesso-dati-pagina]]), la riga sulla lingua dei campi
([[rina-lingua-campi-dynamics]]) e il ramo libero di SCOPE
([[rina-blocco-language-ambito]]). Lo schema e' sempre lo stesso: non manca un divieto, c'e'
un'istruzione che ordina il contrario.

### Perche' e' coerente con quello che si vede

Con interfaccia `en-US`, `CurrentUserLanguage` vale inglese e viene passata al figlio **come
lingua della risposta**. Il figlio obbedisce. La frase aggiunta stasera rafforza il blocco
LANGUAGE abbastanza da vincere su italiano, spagnolo e cinese, ma **non sul francese**.

## Non ho toccato niente

Sono quattro pubblicazioni in un giorno e il difetto va capito prima: queste variabili sono
**dell'orchestratore Microsoft in soluzione gestita**, non nostre, e vanno guardate in
*Variables* prima di decidere. Un errore qui si propaga a tutte le lingue, non solo al francese.

## Da fare domani

1. **Aprire le due variabili** nel pannello Variables: vedere chi le valorizza e chi le legge
   (il padre le passa al figlio? il figlio le usa nel prompt?).
2. Se sono nostre e sono modificabili: **allineare le descrizioni** alla regola vera - la lingua
   viene dal messaggio, non dal contesto F&O.
3. Solo dopo, rimisurare le cinque lingue.

Due variabili per la stessa cosa (`CurrentUserLanguage` e `UserLanguage`) sono gia' di per se'
un segnale: vale la pena capire se servono entrambe.
