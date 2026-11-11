---
name: rina-loop-contabilizzazione-servizi
description: InfiniteLoopDetected quando si chiede la contabilizzazione di un articolo che non ne ha; il figlio non chiude il turno e il padre ridelega
metadata:
  type: project
---

**Riprodotto il 06/10/2026.** Sequenza che lo innesca, in tre messaggi:
1. "Ciao mi potresti dire il materiale che sostituisce Npeste411" → risponde `SE354004.0001` (servizio, categoria Travel)
2. "come posso utilizzare il materiale SE354004.0001" → risponde (in inglese, vedi sotto)
3. "che tipo di contabilizzazione ha quel materiale" → **InfiniteLoopDetected** dopo ~3 minuti

**Cosa succede davvero.** Nella traccia del pannello di test il padre delega al figlio con
`Task = "Provide the accounting/posting type information for material SE354004.0001"`, e il figlio
**non restituisce mai** `resolvedResult` ne `answered`. Il padre non ha nulla da consegnare e
ridelega, finche la piattaforma tronca il turno. Nel frattempo il flusso
`RINA - Get Item Accounting Types` viene invocato in cerchio: **10 esecuzioni nel solo minuto
dell'errore**, tutte riuscite in ~400 ms. Il flusso non e lento ne rotto: e l'agente che lo
richiama. Il `FlowActionTimedOut` che compare qualche minuto prima e lo stesso fenomeno piu lieve.

Ipotesi sul perche: per quell'articolo (un servizio) il flusso non restituisce tipi di
contabilizzazione, e il figlio non ha una regola per "lo strumento non ha restituito nulla" su
questo percorso, quindi insiste. Il flusso e in **Express mode**, che non conserva input e output:
per vederlo va disattivato temporaneamente.

**Difetto separato emerso dalla stessa traccia:** il padre passa `UserLanguage = en-US` anche
quando l'utente scrive in italiano. E la causa delle risposte in inglese: non e il figlio che
sbaglia lingua, e l'orchestratore che gli dice la lingua sbagliata.

**Corretto il 06/10/2026.** Regola aggiunta in `OUTPUT BEHAVIOR` delle istruzioni del figlio:
chiudere sempre il turno valorizzando `answered`, chiamare ogni strumento una sola volta per
articolo, e trattare il risultato vuoto come la risposta invece di riprovare. Rifatta la stessa
sequenza di tre messaggi: nessun loop, risposta in ~70 secondi
("Per l'item SE354004.0001 non risultano tipi di contabilizzazione configurati"). Conferma anche
l'ipotesi: quell'articolo ha davvero zero tipi di contabilizzazione.

**Residuo cosmetico:** nella risposta compare `AccountingTypeCount = 0`, un nome di campo interno
che non dovrebbe arrivare all'utente.

**Why:** e il primo errore bloccante visto da un tester (Enrico, 06/10 11:44 UTC) e si presenta
come "il bot non parla piu".
**How to apply:** la correzione va cercata nelle istruzioni del figlio - una sola chiamata per
articolo, e se lo strumento non restituisce contabilizzazioni dirlo e chiudere il turno con
`answered`. Vedi [[rina-doppio-messaggio-architettura]] per il punto di consegna fra i due agenti.

## 21/10/2026 - il sintomo e' cambiato, poi e' sparito

**Alle 12:25** il caso si e' ripresentato **in forma diversa**: non piu' `InfiniteLoopDetected`
con il flusso rilanciato piu' volte, ma **una sola run riuscita in 41 secondi** e l'orchestratore
fermo su "Operazione in corso" per oltre cinque minuti, senza consegnare niente all'utente.
Quindi il flusso faceva il suo lavoro e la risposta si perdeva a valle.

**Alle 15:00, due prove su due, risponde.** Run di 55 secondi, una sola esecuzione, risposta
completa. Stessa domanda, stessa societa' (RSSE).

**Non ho aperto una segnalazione**: non si apre un item per un difetto che non si riproduce.
Resta da risorvegliare.

Fra i due momenti sono stati pubblicati: il fix delle traduzioni in parallelo
([[rina-traduzioni-LE-costano-20s]]), le due modifiche alle istruzioni dell'orchestratore
([[rina-non-chiedere-la-lingua]] e la riga CAPEX/OPEX) e la correzione del prompt dell'agente
connesso. **Quale dei quattro abbia sbloccato il turno non e' dimostrato.**

**Se ricompare**, il segno da cercare e' questo: una sola run del flusso, riuscita, e l'agente che
non chiude il turno. Non e' il loop vecchio, e guardare le run del flusso non basta - va guardato
il Monitor di Copilot Studio.
