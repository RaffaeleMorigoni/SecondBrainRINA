---
name: rina-template-risposta-ricerca
description: "Il formato approvato per le risposte di ricerca articoli: titolo markdown, elenco numerato uno per riga, poi raccomandazione e domanda"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-08T16:35:16.350Z
---

Template deciso con Raffaele l'08/10/2026 guardando due risposte affiancate in F&O: gli piace la
versione con le intestazioni evidenziate, **piu' una riga di stacco sotto ciascuna**. E' il
formato da mantenere per tutte le ricerche articolo.

**Struttura, per ciascuno dei due blocchi:**
1. un titolo sulla propria riga, nella lingua dell'utente;
2. uno stacco;
3. un elenco numerato con **un articolo per riga**: codice, descrizione ufficiale, tipo;
4. dopo i due blocchi, uno stacco, poi la raccomandazione e la domanda all'utente.

Primo blocco: articoli rilasciati nella societa dell'utente. Secondo: articoli esistenti in
Dynamics da richiedere. Il blocco resta anche quando e' vuoto, con una riga che dichiara l'assenza.

**Due cose imparate scrivendolo:**

*"Lascia una riga vuota" non funziona.* Chiesto cosi', il modello scriveva **un solo `\n`**:
verificato leggendo il testo della risposta nel DOM, dove fra titolo e contenuto c'era un a capo
singolo. La formulazione che funziona e' far scrivere un **titolo markdown** (`###` piu' il
titolo): lo stacco lo produce il renderer, non il modello. Il risultato e' anche piu' leggibile,
perche' il titolo esce piu' grande.

*Senza dirlo, gli articoli escono come un paragrafo unico.* Riversava l'output grezzo dello
strumento separato da `|`, tutto di seguito. Serve l'istruzione esplicita: elenco numerato,
**ONE ITEM PER LINE**, e il divieto di riusare i separatori dell'output dello strumento.

**Dove sta:** sezione `ANSWER FORMAT - MANDATORY` delle istruzioni dell'agente. Attenzione al
limite di 8.000: per far entrare il template ho tolto la lista fissa delle lingue supportate
(che violava comunque [[copilot-studio-language-rule]]) e compresso due frasi. Vedi
[[copilot-studio-istruzioni-limite-8000]].
