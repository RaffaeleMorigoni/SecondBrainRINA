# Le sei lingue, e una configurazione mai guardata

30/10/2026 sera. Stessa domanda — *"quale materiale ha sostituito questo codice Npimmo322?"* —
posta in sei lingue.

| domanda | lingua della risposta | esito |
|---|---|---|
| italiano | **inglese** | **sbagliata** |
| francese | francese | corretta |
| inglese | inglese | corretta |
| spagnolo | **inglese** | **sbagliata** |
| portoghese | **inglese** | **sbagliata** |
| cinese | **inglese** | **sbagliata** |

**Quattro su sei sbagliate, tutte in inglese.**

## Due precisazioni che cambiano l'inquadramento

**1. Non serve sapere la lingua di interfaccia per qualificarlo come difetto.** Le istruzioni
dicono gia': *"Ignore the application interface language and any system setting whenever they
differ from it"*. La lingua la da' la domanda. Anche con locale inglese, quelle risposte
**dovevano** uscire in italiano, spagnolo, portoghese e cinese. Il locale spiegherebbe il
meccanismo, non giustifica il risultato.

**2. Queste domande non passano dalla ricerca.** Nessuna run di `RINA - Search Items` nell'orario:
`Npimmo322` e' un codice legacy, la risposta viene dal **mapping in Knowledge**. Quindi il difetto
e' nell'orchestrazione, non nel flusso.

## Il caso italiano non si riproduce

Stessa domanda, stesso momento, mia interfaccia **italiana**, RSSE:

```
MMD | Il NEW MMD corrispondente e' SE101001.0001 - Manutenzione Edile Ordinaria
Contabilizzazioni consentite:
  1 - I - Costo OPEX indiretto - conto 411509, Manutenzione beni immobili - categoria progetto Mainten
  3 - I - Investimento CAPEX - conto 111600, Transitorio immobilizzazioni materiali in corso
```

Italiano pieno, formato corretto.

## La scoperta: Primary language dell'agente = en-US

In **Settings > Languages** dell'orchestratore:

```
Primary language: English (United States) (en-US)
```

con una lista di **lingue secondarie** (arabo, cinese, danese, olandese, finlandese, francese,
tedesco, greco, italiano…) tutte modificate il **20/10/2026 alle 20:19**, cioe' da qualcun altro
e prima che iniziassimo a lavorare sulla lingua.

**E' una variabile che non abbiamo mai considerato in tutta la giornata**, e che compete con le
istruzioni: l'inglese non e' un ripiego casuale del modello, e' la **lingua primaria configurata
dell'agente**. Coerente con quattro risposte su sei in inglese.

## Cosa NON ho fatto, e perche'

**Non ho toccato la Primary language.** Non so che effetti abbia sui topic localizzati e sulle
secondarie configurate il 20/10 da altri, e oggi ho gia' pubblicato tre modifiche. Cambiarla
a quest'ora, senza una prova controllata, sarebbe esattamente l'errore che oggi mi e' costato
due diagnosi sbagliate.

## Da fare domani, in quest'ordine

1. **Chiedere a Chiara chi ha configurato le lingue il 20/10 e perche'** — le secondarie non le
   abbiamo messe noi.
2. **Rifare le sei prove a freddo**, dichiarando la lingua di interfaccia di chi prova, per avere
   una baseline pulita.
3. Solo allora valutare la Primary language, **una variabile alla volta**, misurando prima e dopo.

Prove: `prove_uat/20261030_primary_language_en-US.png`,
`prove_uat/20261030_Npimmo322_IT_risponde_in_italiano.png`.

---

## Provato a mettere italiano come primaria: **non e' possibile**

Aperto l'editor della Primary language. Il menu offre **quattro sole voci**, tutte varianti
dell'inglese:

```
English (en-US)  -  English (Australia) (en-AU)  -  English (New Zealand) (en-NZ)  -  English (United Kingdom) (en-GB)
```

**L'italiano non e' selezionabile**, pur essendo presente fra le secondarie (`Italian (Italy)
(it-IT)`). La lingua primaria si fissa alla creazione dell'agente e si puo' cambiare solo
all'interno della stessa famiglia: `Copilot for finance and operations apps` e' un agente di
Microsoft in soluzione gestita, nato in inglese.

Annullato senza toccare nulla: la primaria resta `en-US`.

## Cosa cambia nel ragionamento

**L'inglese e' strutturale nell'agente e non e' rimovibile.** Non e' una scelta che possiamo
revocare: e' la lingua di nascita di un agente Microsoft che non abbiamo creato noi.

Ne segue che **l'unica leva sulla lingua resta `UserLanguage` e le istruzioni** - cioe' esattamente
il terreno su cui si e' lavorato tutto il giorno. Il difetto non e' "hanno configurato male la
lingua", e' "la lingua dell'utente deve vincere su una primaria inglese che non si puo' cambiare".

E rende piu' comprensibile perche' l'inglese riaffiori: non e' un ripiego arbitrario del modello,
e' il default della piattaforma sotto le istruzioni.

## Resta da chiedere a Chiara

Chi ha aggiunto le **nove e piu' lingue secondarie** il 20/10 alle 20:19, e con quale scopo:
servono alla localizzazione dei topic, non alla generazione, e non sappiamo se qualcuno conti
su di esse.
