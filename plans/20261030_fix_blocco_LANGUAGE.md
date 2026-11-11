# Blocco LANGUAGE rafforzato — misurato prima e dopo

30/10/2026, orchestratore `Copilot for finance and operations apps`. Pubblicato alle **18:30**.

## Metodo

Baseline **prima** della modifica, poi modifica, poi le stesse domande. Interfaccia F&O portata a
**en-US**, cioe' la condizione in cui il difetto si manifesta; sessione nuova a ogni prova;
societa' RSSE; domanda sempre la stessa (`Npimmo322`), cambia solo la lingua.

La mia interfaccia e' stata **riportata a `it`** a fine prove.

## Risultato

| domanda | PRIMA | DOPO |
|---|---|---|
| italiano | italiano — corretta | **italiano — corretta** (nessuna regressione) |
| spagnolo | **inglese — sbagliata** | **spagnolo — corretta** |
| cinese | **inglese — sbagliata** | **cinese — corretta** |

Due difetti su due risolti, nessuna regressione sul caso che gia' funzionava.

Esempi del dopo:

```
ES  MMD | El material actual es SE101001.0001 - Mantenimiento ordinario de edificios…
        1 - I - Coste OPEX indirecto [Costo OPEX indiretto] - cuenta 411509,
        Mantenimiento de inmuebles [Manutenzione beni immobili]

ZH  MMD | …SE101001.0001 - 建筑日常维护 (Manutenzione Edile Ordinaria)
        1 - I - 间接 OPEX 成本 (Costo OPEX indiretto) - 总账科目 411509,
        房地产维护 (Manutenzione beni immobili)
```

## La modifica: una frase

Nel blocco `LANGUAGE`, dopo *"Never let system locale, previous turns or tool-result language
override it."*:

```
It binds also free answers, Knowledge answers and the agent's own default language.
```

83 caratteri. Istruzioni da **7.889 a 7.973 / 8.000** - restano 27 caratteri. Verificato dopo il
salvataggio che la coda non fosse troncata ([[copilot-studio-istruzioni-limite-8000]]).
**Nessuna lingua nominata**, come da regola.

## Perche' funziona: ancora un permesso, non un divieto mancante

Il blocco LANGUAGE era gia' forte (*"authoritative"*, *"never let system locale… override it"*).
Mancavano due percorsi:

1. **Il ramo libero di SCOPE**: *"For any other request … behave freely, **as the standard Copilot
   for finance and operations apps**"*. Lo standard di quell'agente e' `en-US`
   ([[rina-primary-language-agente]]).
2. **Le risposte da Knowledge**: le domande su codici legacy come `Npimmo322` non passano dagli
   strumenti, quindi *"tool-result language"* non le copriva.

**Terza volta oggi che la causa e' un permesso e non un divieto mancante**, dopo il 2418 e la riga
sulla lingua dei campi. La frase non vieta: estende l'ambito della regola gia' esistente.

## Cosa resta aperto

- Le **lingue secondarie** configurate il 20/10 da altri: da chiedere a Chiara.
- Una **prova a freddo domani** sugli stessi tre casi: oggi la pubblicazione e' recente.
- Portoghese e francese non li ho rimisurati: stasera francese era corretto e portoghese sbagliato
  nelle prove dell'utente. Da includere nella prova a freddo.
- In una risposta spagnola compare *"sociedad RSPA"* mentre la sessione era su **RSSE**: da
  verificare, e' il tema [[rina-company-sessione-dataareaid]].

Ripristino: `backup/20261030_ROLLBACK_padre_blocco_LANGUAGE.md`.
Prove: `prove_uat/20261030_PRIMA_ES_*`, `_DOPO_ES_*`, `_PRIMA_ZH_*`, `_DOPO_ZH_*`.

---

## Portoghese e francese, aggiunti dopo

Stesse condizioni: interfaccia `en-US`, sessione nuova, RSSE, stessa domanda.

| domanda | lingua della risposta | contenuto |
|---|---|---|
| portoghese | **portoghese** — corretta | **incompleto**: non da' il codice |
| francese | **inglese** — sbagliata | corretto nel merito |
| francese (ripetuta) | **inglese** — sbagliata | corretto nel merito |

### Portoghese: lingua risolta, contenuto no

> *MMD | O mapeamento para o NEW MMD foi localizado, mas o material atual ainda precisa ser
> validado no Dynamics antes de ser informado.*

La lingua e' giusta. Ma l'agente **trova il mapping e non lo comunica**, dicendo che va prima
validato in Dynamics. E' coerente con la regola *"Always run a search first"* introdotta il 28/10
([[rina-permesso-dati-pagina]]): ha il dato dalla Knowledge e si trattiene perche' non l'ha
verificato. Per il tester resta una mancata risposta.

### Francese: due prove su due in inglese

> *MMD | The current NEW MMD is SE101001.0001 — Ordinary building maintenance. It is released in
> your RSSE company.*

**Non e' intermittenza**: ripetuto, stesso esito. Ed e' l'unica lingua delle cinque a fallire.

**Non posso chiamarla regressione con certezza**: nelle prove dell'utente il francese era
corretto, ma con lingua di interfaccia ignota, quindi non e' un confronto valido. Quello che so:
**con interfaccia `en-US`, dopo la modifica, il francese risponde in inglese, due volte su due.**

## Quadro complessivo dopo la modifica (interfaccia `en-US`)

| lingua | lingua risposta | note |
|---|---|---|
| italiano | italiano | corretta |
| spagnolo | spagnolo | corretta — era inglese |
| cinese | cinese | corretta — era inglese |
| portoghese | portoghese | corretta nella lingua, contenuto incompleto |
| francese | **inglese** | **unica lingua ancora rotta, 2 prove su 2** |

**Quattro lingue su cinque corrette nella lingua.** Il francese resta, e va capito prima di
dichiarare chiuso il tema.

## Da fare, nell'ordine

1. **Il francese a freddo domani**, con baseline esplicita: prima con l'istruzione attuale, e se
   serve confrontare togliendo la frase aggiunta - per sapere se e' regressione o difetto
   preesistente mai misurato in questa condizione.
2. **Il portoghese e' un altro tema**: non lingua ma contenuto, e tocca la regola sulla
   validazione in Dynamics. Da non mischiare.

Prove: `prove_uat/20261030_DOPO_PT_lingua_ok_contenuto_incompleto.png`,
`prove_uat/20261030_DOPO_FR_risponde_in_inglese.png`.

---

## A/B sul francese: non e' una regressione

Per distinguere regressione da difetto preesistente ho **tolto la frase**, ripubblicato
(istruzioni tornate a 7.889) e rifatto la prova francese nelle stesse condizioni
(interfaccia `en-US`, sessione nuova, RSSE).

| | francese |
|---|---|
| **con** la frase | inglese (2 prove su 2) |
| **senza** la frase | **inglese** |

> *…historical code provided is SE101001.0001 — Routine building maintenance (Manutenzione Edile
> Ordinaria). It is a service, available in your RSSE company…*

**La frase non c'entra.** Il francese risponde in inglese in entrambe le configurazioni: e' un
difetto **preesistente**, mai misurato prima in questa condizione perche' la lingua di interfaccia
non era mai stata dichiarata nelle segnalazioni.

**Frase rimessa e ripubblicata** (7.973 caratteri, verificata): migliora spagnolo e cinese, sul
francese non cambia nulla, quindi resta.

Interfaccia F&O riportata a `it`.

## Stato finale della serata

| lingua | lingua risposta |
|---|---|
| italiano | italiano |
| spagnolo | spagnolo |
| cinese | cinese |
| portoghese | portoghese (contenuto incompleto, tema a parte) |
| **francese** | **inglese — aperto, non spiegato** |

Il francese e' l'unico caso rimasto, ed e' **anomalo**: e' l'unica lingua che fallisce, e falli-
sce in modo stabile. Da indagare a freddo - va escluso qualcosa di specifico al francese
(es. una delle lingue secondarie configurate il 20/10, dove `French (France) (fr-FR)` compare).

Prova: `prove_uat/20261030_FR_senza_la_frase_ancora_inglese.png`.
