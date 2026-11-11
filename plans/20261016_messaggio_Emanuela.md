# Messaggio per Emanuela — 16/10/2026

Ciao Emanuela,

ho appena pubblicato una modifica che riguarda proprio quello che stai vedendo con Enrico.

**Sulla tua domanda delle voci miste.** Hai ragione a trovarla strana, ma non era l'agente a
essere incoerente: erano i dati. In anagrafica alcuni codici hanno la descrizione italiana gia'
scritta in inglese — per esempio SE302501.0001 ha come descrizione italiana "Software projects
development". Per questo quella voce ti appariva in inglese mentre le altre due no: fino a
stanotte l'agente restituiva sempre e solo il campo italiano, qualunque fosse la lingua della
conversazione.

**Cosa e' cambiato.** Ora l'agente legge le traduzioni ufficiali delle descrizioni che RINA ha
caricato in Dynamics: esistono in italiano, inglese, francese, portoghese, spagnolo e cinese, con
copertura piena sui codici MMD nuovi. Non sono traduzioni automatiche, sono i testi ufficiali.
Inoltre ora anche le frasi dell'agente seguono la lingua di chi scrive: prima restavano in
italiano perche' l'orchestratore aveva l'istruzione di non tradurre la risposta.

Il risultato e' che una domanda in francese riceve una risposta interamente in francese, codici e
descrizioni compresi. Se rifai la tua prova in inglese, le tre voci ora dovrebbero essere tutte
in inglese (Application development and configuration / Personnel training and development
consulting).

**Due avvertenze per i test di adesso.**

1. Conviene **chiudere e riaprire la finestra di Copilot**: ho pubblicato da pochi minuti e le
   conversazioni gia' aperte possono ancora usare la versione precedente.

2. **Attenzione alla societa'.** La ricerca lavora sulla legal entity della sessione F&O, quella
   che si vede nella barra in alto. Se Enrico sta su **RSPA** non trovera' quasi nulla: in UAT
   l'anagrafica dei nuovi MMD li' e' praticamente vuota, e non e' un difetto dell'agente.
   Per provare davvero conviene passare a **RSSE**, dove i materiali ci sono.
   Se nel selettore la societa' non compare, l'utenza va abilitata: serve un amministratore
   (Amministrazione sistema > Utenti > Ruoli utente > Assegna organizzazioni).

**Una cosa da sorvegliare.** Per ottenere la traduzione ho allentato il vincolo che impediva
all'orchestratore di riformattare la risposta. Nei test l'elenco e' rimasto corretto, uno per
riga, ma se doveste vedere le voci schiacciate in un unico paragrafo segnalatemelo subito:
si torna indietro in un minuto.

Fammi sapere cosa vedete, cosi' lo verifichiamo subito.
