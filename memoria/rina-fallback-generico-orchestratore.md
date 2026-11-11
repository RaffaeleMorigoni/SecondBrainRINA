---
name: rina-fallback-generico-orchestratore
description: "Il \"Mi dispiace, non so come posso aiutarti\" compare solo dentro Evaluation: nel canale reale le stesse domande ricevono la risposta giusta"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-07T15:36:01.584Z
---

**E' un artefatto della suite, non un difetto dell'agente.** Nella run del 07/10/2026 su 46 casi,
6 degli 11 fallimenti erano la risposta
**"Mi dispiace, non so come posso aiutarti. Puoi provare a riformulare la richiesta?"**, senza
prefisso `MMD | ` e senza che nessuno strumento venisse invocato. Avevo concluso che fosse il
difetto piu' grave dell'agente: **sbagliato**.

La stessa domanda posta nel **pannello Copilot dentro F&O** riceve la risposta corretta. Per
`MMD code che e collegato al conto di contabilita 411311`:
"MMD | L'elenco dei codici MMD non puo' essere prodotto partendo dal solo conto contabile 411311.
Fornire un codice MMD, una categoria oppure una descrizione del materiale/servizio da verificare."
Prefisso corretto, limite dichiarato in una frase, alternative indicate: e' l'atteso.

**Le famiglie che in Evaluation cadono nel fallback:** domande sul comportamento dell'agente
("puoi creare tu la RDA?", "scegli tu la contabilizzazione", "compilami il draft"), domande
contabili senza articolo in mano (conto `411311`, tipo `3 Capex`, soglie AFC), e le alternative
contabili chieste in modo discorsivo. Nessuna e' una ricerca di articolo.

**Ipotesi da verificare** (non ancora confermata): in Evaluation l'agente viene invocato senza il
contesto che in F&O arriva dal canale — le istruzioni dichiarano di ricevere `Task`,
`GlobalLegalEntity` e `UserLanguage`, e il profilo utente del test set potrebbe non popolarli
tutti. Da controllare prima di attribuire a questi casi un qualunque significato.

**Come applicarlo:** un fallimento della suite su una domanda che non e' una ricerca di articolo
va **riprodotto nel pannello di test o in F&O prima di toccare qualsiasi cosa**. Se li' la
risposta e' corretta, il caso misura il banco di prova e non l'agente: correggere l'atteso o
togliere il caso, non le istruzioni. Vale anche al contrario: il 07/10 ho aggiunto una regola
alle istruzioni per il caso dell'attributo contabile e il punteggio e' peggiorato
(83/66 -> 79/62) senza che il comportamento reale fosse rotto. Vedi [[rina-suite-regressione]].

**09/10/2026 - il marcatore di citazione `[1]` non si toglie dalle istruzioni.** Avevo aggiunto
*"Never end an answer with a bare number, a citation marker or any symbol you cannot explain"* per
chiudere la riga 20 del registro UAT, e dopo due prove in F&O avevo detto a Raffaele che era
rientrato. **Sbagliato: nella suite del 09/10 dieci risposte su 50 lo contengono ancora.** Il
marcatore non lo scrive il modello, lo inserisce il motore delle citazioni sulle fonti di
conoscenza: nessuna istruzione lo raggiunge. In Copilot Studio non c'e' un interruttore nelle
impostazioni dell'agente (controllate Generative AI e Advanced); va cercato sulla configurazione
delle singole fonti.

**How to apply:** prima di dichiarare chiuso un difetto di forma, contarlo su tutta la suite, non su
due prove in F&O. Due prove non distinguono "corretto" da "non capitato".
