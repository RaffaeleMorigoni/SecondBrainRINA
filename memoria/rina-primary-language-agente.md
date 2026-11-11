---
name: rina-primary-language-agente
description: "Primary language = en-US e NON si puo' cambiare in italiano: il menu offre solo varianti inglesi. L'inglese e' strutturale nell'agente, l'unica leva resta UserLanguage"
metadata:
  type: project
---

**Scoperto il 30/10/2026**, dopo una giornata passata a cercare la causa della lingua nelle
istruzioni.

In **Settings > Languages** di `Copilot for finance and operations apps`:

```
Primary language: English (United States) (en-US)
```

con una lista di **lingue secondarie** (arabo, cinese, danese, olandese, finlandese, francese,
tedesco, greco, italiano…) **modificate il 20/10/2026 alle 20:19 - non da noi**.

**Perche' conta**: quando le risposte escono in inglese, non e' il modello che "ripiega": e' la
lingua primaria configurata dell'agente, che compete con le istruzioni. Le stesse istruzioni che
dicono *"Ignore the application interface language"* non parlano di questa impostazione.

**Il caso che l'ha fatta emergere**: la stessa domanda in sei lingue, **quattro risposte su sei
in inglese** (italiano, spagnolo, portoghese, cinese), con francese e inglese corretti.

**Provato a metterla in italiano: NON si puo'.** Il menu offre quattro sole voci, tutte inglesi
(`en-US`, `en-AU`, `en-NZ`, `en-GB`). La primaria si fissa alla creazione e
`Copilot for finance and operations apps` e' un agente Microsoft in soluzione gestita, nato in
inglese. Annullato senza toccare nulla.

**Conseguenza**: l'inglese e' **strutturale e non rimovibile**. L'unica leva sulla lingua resta
`UserLanguage` e le istruzioni. Il difetto va posto cosi': *la lingua dell'utente deve vincere su
una primaria inglese che non si puo' cambiare* - non come una configurazione sbagliata.

Resta da chiedere a Chiara chi ha aggiunto le secondarie il 20/10 e a cosa servono.

**Correzione a me stesso**: avevo chiesto la lingua di interfaccia di chi provava. Non serve per
qualificare il difetto - **la lingua la da' la domanda**, e le istruzioni lo impongono gia'
([[rina-lingua-interfaccia-fo]] spiega il meccanismo, non giustifica il risultato).

Dettaglio in `plans/20261030_sei_lingue_e_primary_language.md`.
