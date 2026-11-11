---
name: rina-knowledge-compete-con-la-ricerca
description: "I file Excel dei codici in Knowledge facevano rispondere l'agente senza interrogare F&O: causa vera del difetto 2418"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-21T14:28:26.785Z
---

**La scoperta del 21/10, ed e' la causa vera del 2418.** Nella Knowledge dell'orchestratore ci
sono due file caricati a inizio progetto:

- `RINA_MD_Material_Codes_v1 (2) (1).xlsx`
- `Old_new_only_RINA (1) (1).xlsx`

L'agente **pescava i codici da li' invece di chiamare `RINA - Search Items`**. Il segno da cercare
e' il marcatore di citazione **«1»** in fondo alla risposta.

**Come l'ho dimostrato.** Domanda in francese su RSSE: risposta
`SE103003.0001 – Naval mechanical systems maintenance [Entretien des installations…]`, e **nessuna
run** nel flusso. Facendo poi verificare quel codice dal catalogo, il flusso parte e risponde che
**in RSSE non e' rilasciato: e' di PATR**.

**Il danno vero non era la lingua.** Dava a un utente RSSE un codice **non utilizzabile nella sua
societa'**, presentandolo come la risposta. L'inglese con la traduzione fra parentesi era solo il
sintomo: la descrizione veniva dal file, non dal catalogo, e i file non sanno nulla dei rilasci
per societa'.

**I file non si possono togliere**: servono per il mapping vecchio-nuovo, vedi
[[rina-knowledge-mapping-oldnew]]. Toglierli riaprirebbe il difetto dei codici legacy.

**Cosa ha funzionato.** Una regola generica ("never name a code from your own knowledge") **non ha
avuto effetto**: stessa risposta, nessuna run. Ha funzionato invece **nominare il conflitto**,
dicendo a cosa servono quei file, nel blocco `SCOPE` dell'orchestratore:

```
The material code files in Knowledge are for legacy-to-new mapping only, never a list of usable codes: run a search before naming any code, including one just read there.
```

Dopo: `Aucun MMD actif correspondant... n'a ete trouve avec cette recherche`, in francese, senza
parentesi, senza inventare codici.

**La lezione**: quando una fonte alternativa e' disponibile, vietarla genericamente non basta.
Va detto **a cosa serve** e a cosa no.

**Resta aperto**: sull'approfondimento (secondo turno della stessa conversazione) ricompare la
domanda sulla lingua - vedi [[rina-non-chiedere-la-lingua]], che regge sul primo turno ma non sul
follow-up.

## Rettifica del 21/10 sera

Attenzione: nel pannello di test la fonte delle descrizioni inglesi **non era la Knowledge, era il
server MCP** (vedi [[rina-mcp-dynamics-solo-nel-test]]). In **F&O** pero' l'MCP non e' connesso e
non viene mai invocato, quindi li' la correzione descritta sopra resta valida e verificata.
Il peso della Knowledge come fonte alternativa **in F&O** va considerato dimostrato solo dal
cambiamento di comportamento osservato dopo la modifica, non dal log del pannello di test.
