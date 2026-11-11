---
name: rina-casi-dafne-alba
description: "La tabella risultato-vs-atteso del 10/11 non e' confrontabile: i casi sono anteriori al cambio modello del 30/10 e due su sei, rigiocati, danno esito diverso"
metadata:
  type: project
---

**Tabella di Chiara del 10/11/2026**, sei casi segnalati da Dafne e Alba (codici restituiti
dall'agente vs attesi). Verificata in anagrafica e in parte riprodotta.

**I sei casi non sono la stessa cosa:**

| caso | natura |
|---|---|
| Supporto tecnico onsite | **difetto vero**: nessun articolo contiene "onsite", l'agente ne propone 4 |
| Project Planning | **difetto vero + buco tassonomia**: esistono solo codici legacy |
| Bottiglie plastica/vetro | **equivoco semantico**: l'agente prende *vetro* (MA101019 = prodotti in vetro), l'utente voleva l'uso (MA101037 = consumo per laboratori) |
| Resistenza impianti pilota | **convenzione aziendale**: agente propone un bene (ME201003 strumenti), atteso un servizio (SE153002 manutenzione) |
| Training | **segnalazione imprecisa**: `SE702502` era proposto in **2 esecuzioni su 3** |
| Ferro per fonderia | da verificare, dipende dalla societa' (RCSM) |

**Due motivi per cui la tabella non e' una base affidabile:**

1. **Le risposte non sono deterministiche** ([[rina-keyword-non-deterministiche]]): rigiocati
   oggi, *onsite* da' quattro codici **diversi** e *Project Planning* risponde **correttamente*.
2. **I casi sono anteriori al 9/10**, quindi prodotti con **GPT-5.5 Chat**, ritirato il 30/10
   ([[rina-ritiro-gpt55-30-ottobre]]). Non e' lo stesso agente.

**Da chiedere sempre, per i prossimi casi**: societa' (legal entity), **testo esatto digitato**
(non il riassunto) e **almeno 2-3 ripetizioni** per caso. E ricordare che il monitoraggio tiene
le conversazioni **29 giorni**: oltre, la sessione originale non si recupera piu'.

Dettaglio in `plans/20261110_casi_Dafne_Alba.md`, bozza di risposta in
`plans/20261110_risposta_Chiara_Emanuela.md`.
