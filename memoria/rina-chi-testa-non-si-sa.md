---
name: rina-chi-testa-non-si-sa
description: "L'identita' di chi usa l'agente non e' ricavabile: Analytics da' solo numeri, gli input del trigger danno InternalServerError e nessuna azione del flusso usa UserEmail"
metadata:
  type: project
---

**Provate il 06/11/2026 tutte le strade, tutte chiuse:**

1. **Copilot Studio > Monitor > Sessions**: il CSV ha SessionId, data, esito, turni,
   trascrizione, primo messaggio, topic, canale, CSAT. **Nessun campo utente.**
2. **Power Automate, `UserEmail` del trigger**: il pannello della run dice *"Error loading
   inputs"* e l'API risponde `InternalServerError`. Su tutte le run, non e' un caso isolato.
3. **Azioni del flusso**: le 35 azioni sono tutte su prodotti, traduzioni e categorie.
   **Il flusso riceve `UserEmail` ma non lo usa mai**, quindi non compare in nessun input.
4. **Monitor > Activity > Active users**: solo grafici, mai nomi.

**Quello che si ottiene invece, ed e' utile:**
- **Average DAU** (utenti attivi al giorno) e il grafico giornaliero, che da' il **numero di
  utenti distinti per giorno**;
- il volume di sessioni per giorno, scaricabile un giorno alla volta (retention **29 giorni**);
- la **lingua e il contenuto** della prima domanda, che distingue le nostre prove da quelle dei
  tester.

**Cautela**: il MAU dell'orchestratore (ottobre ~230) **non sono i tester RINA**:
`Copilot for finance and operations apps` e' il Copilot generale di F&O e conta anche chi lo usa
per altro. Affidabile solo il dato giornaliero recente, incrociato con i messaggi.

**Se servono i nomi: chiederli a RINA.** Noi non li abbiamo.

Dettaglio e numeri in `plans/20261106_chi_sta_testando.md`.
