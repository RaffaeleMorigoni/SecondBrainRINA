---
name: rina-currentuserrequest-non-aggiornato
description: "Al secondo turno il padre passa all'agente connesso la domanda del turno precedente: causa del 2418 e della lingua persa sul follow-up"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T14:36:11.644Z
---

**La causa, dimostrata il 21/10 nel pannello di test** (in F&O il Monitor non registra i turni:
tutte le sessioni a 0 step, quindi va riprodotto li').

Confronto degli input passati all'agente connesso:

| turno | `CurrentUserRequest` passato |
|---|---|
| 1° — *"Quels codes MMD existent pour l'entretien des installations mécaniques navales ?"* | la stessa domanda, corretto |
| 2° — *"Il s'agit de la maintenance des moteurs et des systèmes de propulsion…"* | **ancora la domanda del 1° turno** |

**Il messaggio nuovo non viene passato affatto.** Anche `Task` resta quello del primo turno.
`UserLanguage` invece e' corretto (`fr`) in entrambi.

**Perche'**: negli **Inputs** dell'agente connesso `CurrentUserRequest` e' impostato su
**Dynamically fill with AI**, non agganciato al messaggio dell'utente. Al secondo turno il modello
ci rimette la domanda precedente.

**Cosa spiega**: il difetto 2418, il caso di Chiara su PATR (prima risposta giusta, seconda no) e
la domanda sulla lingua che ricompare sul follow-up ([[rina-non-chiedere-la-lingua]]). Il figlio
non ha mai visto il messaggio corrente: non puo' dedurne la lingua ne' il contenuto.

**Il fix naturale** e' passare `CurrentUserRequest` a **Custom value** agganciato al messaggio
corrente. **Non applicato il 21/10**: cambia il contratto padre-figlio (oggi il padre passa una
richiesta gia' elaborata, non il testo grezzo) e va concordato.

**Non perdere tempo con le istruzioni**: il problema non e' cosa l'agente deve fare, e' che non
riceve il dato. Tre regole aggiunte in giornata non hanno toccato questo comportamento.

**Scoperta collaterale**: nel log compare `MCP Dynamics 365 ERP Procurement & MMD` —
"The agent invoked the MCP server". Esiste una seconda via di accesso a F&O oltre ai flussi, che
non passa per le traduzioni ne' per il filtro societa'. Probabilmente e' **questa** la fonte delle
descrizioni inglesi, piu' della Knowledge
([[rina-knowledge-compete-con-la-ricerca]], da rivedere alla luce di questo).

## Il fix e' stato tentato il 21/10 e ANNULLATO: peggiorava

`CurrentUserRequest` portato a *Custom value* = **`System.Activity.Text`**, salvato e pubblicato.
**Risultato: il primo turno e' regredito.** Alla stessa domanda francese **il flusso non e' piu'
partito** (nessuna run) ed e' tornata la descrizione inglese con le parentesi.

**Perche':** passando il **testo grezzo** l'agente connesso perde la richiesta gia' elaborata
dall'orchestratore, che e' quella che fa scattare la ricerca. Il contratto padre-figlio non
prevede il messaggio nudo. **`System.Activity.Text` non e' la strada.**

Annullato e ripubblicato subito nella configurazione originale (*Dynamically fill with AI*).

## Stato finale verificato, entrambi i turni corretti

Dopo il ripristino, due turni consecutivi in francese su RSSE **funzionano**: il secondo risponde
alla domanda **nuova**, in francese, senza chiedere la lingua. E in piu' l'agente dice ora che
`SE103003.0001` **non e' disponibile in RSSE ma in PATR e va richiesto** — l'informazione che
mancava all'origine del 2418.

**Ma attenzione: il comportamento e' cambiato piu' volte in un'ora**, a volte a parita' di
configurazione: la stessa sequenza che funziona falliva 40 minuti prima. Verosimilmente le
pubblicazioni si propagano con ritardo, non dimostrato. **Ripetere a freddo prima di chiudere**, e
non fidarsi di una singola prova riuscita.

## 22/10 — prova a freddo: NON regge

Rifatta la stessa sequenza il mattino dopo, chat pulita su RSSE, **nessuna pubblicazione nella
notte**. Il difetto **c'e' ancora**:

- turno 1: `SE103003.0001 — Naval mechanical systems maintenance [Entretien des installations…]`,
  descrizione **in inglese con parentesi**, e **sparisce** l'avviso che il codice non e'
  disponibile in RSSE;
- turno 2: risponde alla domanda giusta e la prosa e' in francese, ma la descrizione resta inglese;
- **nessuna run del flusso**, su nessuno dei due turni.

**La prova riuscita del 21/10 sera era transitoria.** Verosimilmente legata alla propagazione
delle pubblicazioni. Conferma che **non si chiude un difetto su una prova riuscita subito dopo
una pubblicazione**: va sempre ripetuta a freddo il giorno dopo.

**Cosa resta acquisito**: la prosa esce in francese e il secondo turno non chiede piu' la lingua
(quelli reggono). **Cosa resta rotto**: l'agente non invoca la ricerca, quindi prende le
descrizioni da un'altra fonte in inglese e perde il filtro per societa'.

**Le regole nelle istruzioni non bastano a forzare l'invocazione dello strumento.** Tre tentativi
il 21/10, nessuno ha cambiato questo comportamento.
