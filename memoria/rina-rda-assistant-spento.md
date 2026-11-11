---
name: rina-rda-assistant-spento
description: Le domande "voglio aprire una RDA" non ricevono risposta perche' RDA Creation Assistant e' disattivato nell'orchestratore
metadata:
  type: project
---

Nell'orchestratore `Copilot for finance and operations apps`, scheda **Agents**, ci sono due
figli: `RINA Procurement & MMD Assistant` (On) e **`RDA Creation Assistant` (Off)**.

**Conseguenza:** a una domanda come "vorrei aprire una rda per una consulenza informatica nello
sviluppo dei report" l'orchestratore riconosce l'intento, prova a delegare, non trova nessuno e
chiude il turno con una **parafrasi della domanda**:
`MMD | Richiesta di creazione/apertura di una RDA per consulenza informatica nello sviluppo di report.`
Riprodotto sia nella suite (Fail/Fail, punteggio 25) sia in F&O.

**Dimostrato il 15/10:** riacceso l'agente e ripubblicato, la stessa domanda riceve
*"Per aprire la RDA mi servono il codice progetto, il codice articolo/MMD da utilizzare per la
consulenza informatica, la quantita' e l'importo stimato della riga; il fornitore e' facoltativo."*
Nessuna RDA creata: chiede solo i dati. Poi **rispento e ripubblicato**, come da accordo: lo
stato corrente e' di nuovo Off.

**Da decidere con RINA:** se l'agente RDA deve restare spento in UAT, serve almeno una risposta
sensata ("la RDA si crea in Dynamics") invece della parafrasi; se deve essere acceso, e' una
scelta di perimetro loro. Vedi [[rina-fo-user-worker-prerequisite]] per il prerequisito del
lavoratore collegato all'utenza.
