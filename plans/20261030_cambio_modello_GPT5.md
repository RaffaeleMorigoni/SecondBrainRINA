# Cambio modello GPT-5.5 Chat → GPT-5 Chat — 30/10/2026

**Fatto oggi perche' scadeva oggi.** L'avviso in pagina diceva:
*"This agent uses GPT-5.5 Chat which will retire starting on Oct 30, 2026. If you don't change
the model, we'll switch to a newer one."* — cioe' **se non sceglievamo noi, sceglieva Microsoft**.

## Cosa ho cambiato

| agente | id | prima | dopo | pubblicato |
|---|---|---|---|---|
| `Copilot for finance and operations apps` (padre) | `679d8694…` | GPT-5.5 Chat | **GPT-5 Chat** | 30/10 ore 10:19 |
| `RINA Procurement & MMD Assistant` (standalone) | `dfd051a3…` | GPT-5.5 Chat | **GPT-5 Chat** | 30/10 ore 10:22 |

Gli agenti **connessi** (`b4a1f29d…` figlio, `RDA Creation Assistant`) non hanno selettore di
modello: ereditano dal padre. Censimento del 21/10 confermato.

Scelta `GPT-5 Chat` come da piano: stessa famiglia, non in preview, ed e' gia' il default di
Microsoft per gli altri agenti dello stesso ambiente.

Un warning nuovo e previsto dopo il cambio: *"The model for this agent has changed, but no
evaluation has been run since."* Gli altri due (credenziali autore, premium features) erano gia' li'.

---

# ESITO DELLA SUITE: **il nuovo modello e' peggiore. Regressione netta.**

Confronto con il "prima" rilevato **quaranta minuti prima**, stesse istruzioni, stesso flusso,
stessa interfaccia `it`, stessa societa' RSSE (`20261030_PRIMA_del_cambio_modello.md`).

## Caso 1 — italiano: formato cambiato, prosa sparita

**Dopo:**
```
MMD | SE153002.0001 Manutenzione impianti di laboratorio
MMD | SE153003.0001 Manutenzione microscopi elettronici
MMD | SE153004.0001 Taratura strumentazione di laboratorio
MMD | SE153005.0001 Manutenzione altre apparecchiature specialistiche di laboratorio
```

- ✅ **l'elenco ora va uno per riga** — e' il difetto di formato che inseguivamo da settimane;
- ❌ **il prefisso `MMD | ` e' ripetuto su ogni riga**. Le istruzioni dicono *"once at the very
  beginning"*: e' una violazione secca;
- ❌ **sparita tutta la prosa**: niente frase introduttiva, niente indicazione di quale codice sia
  il piu' pertinente, niente invito a restringere la ricerca. Prima c'erano.

Prova: `prove_uat/20261030_DOPO_cambio_modello_1_IT_prefisso_ripetuto.png`

## Caso 4 — francese: **rotto su entrambi i turni**

**Turno 1** (domanda in francese):
```
MMD | SE103003.0001 Manutenzione impianti meccanici navali
```
**In italiano.** E senza nessuna delle informazioni che c'erano prima: niente "non rilasciato in
RSSE", niente PATR, niente codici alternativi disponibili.

**Turno 2** (domanda nuova, in francese): **identica riga, di nuovo**. Non risponde alla domanda
nuova, ripete la precedente.

Quaranta minuti prima, stesso caso, il vecchio modello rispondeva **in francese, due volte su
due**, con la disponibilita' e le alternative.

Prova: `prove_uat/20261030_DOPO_cambio_modello_4_FR_risposta_italiana_e_ripetuta.png`

---

## Come lo leggo

Tre osservazioni, tutte nella stessa direzione: **il nuovo modello segue le istruzioni molto
meno**. Perde la lingua, perde il contenuto, ripete il prefisso, non distingue il secondo turno
dal primo. L'unica cosa migliorata e' l'a capo dell'elenco.

**Onesta' sul campione**: una prova per caso, e l'agente e' instabile di suo. Ma le tre
osservazioni sono coerenti fra loro e **tutte diverse dal "prima"**, che era stato rilevato lo
stesso giorno. Non e' rumore.

## Cosa si puo' fare, in ordine

1. **Riprovare la suite fra qualche ora.** Dopo una pubblicazione il comportamento si assesta, e
   il 21/10 abbiamo gia' visto prove riuscite sciogliersi e viceversa. Costa nulla.
2. **Provare un altro modello.** `GPT Chat Latest` (stessa famiglia, segue l'ultimo), oppure
   `Claude Sonnet 4.6`. Una alla volta, con questa stessa coppia di casi.
3. **Riadattare le istruzioni al modello nuovo.** E' la via piu' lunga: il prefisso ripetuto e la
   lingua persa sono entrambi governati da righe che il vecchio modello rispettava.
4. **Tornare a GPT-5.5 Chat** e' ancora possibile oggi dal selettore, ma e' in ritiro: ci
   ricompra qualche giorno, non e' una soluzione.

**Da non fare**: toccare le istruzioni oggi. Abbiamo appena cambiato il motore, e mescolare le
due cose rende impossibile capire cosa ha prodotto cosa. Una variabile alla volta.

---

# CONFRONTO FRA TRE MODELLI — 30/10, ore 10:15-11:05

Provati tutti sullo stesso padre, stessi due casi, stessa mattina, istruzioni e flusso invariati.

| | **GPT-5.5** (il vecchio) | **GPT-5 Chat** | **Claude Sonnet 4.6** | **GPT Chat Latest** |
|---|---|---|---|---|
| prefisso `MMD \| ` | una volta ✅ | **su ogni riga** ❌ | **assente** ❌ | **una volta** ✅ |
| elenco a capo | **no** ❌ | si' ✅ | si' (tabella) | **si'** ✅ |
| FR turno 1 | francese completo ✅ | **italiano** ❌ | ottimo ✅ (1 timeout) | francese ✅ |
| FR turno 2 | intermittente (1 su 2 chiedeva la lingua) | **italiano, ripetuto** ❌ | **il migliore di tutti** ✅ | francese ✅ ma *"aucun MMD trouvé"* ❌ |
| stile | prosa | scarno | tabelle, emoji, sezioni | prosa asciutta |

## Come la leggo

**`GPT Chat Latest` e' il migliore sulla forma.** E' l'unico che mette il prefisso una volta sola
**e** manda l'elenco a capo: cioe' il formato che inseguivamo da settimane, finalmente giusto.

```
MMD | Per un servizio di manutenzione delle attrezzature di laboratorio, i risultati
piu' pertinenti disponibili in RSSE sono:
• SE153005.0001 Manutenzione altre apparecchiature specialistiche di laboratorio
• SE153002.0001 Manutenzione impianti di laboratorio
• SE153003.0001 Manutenzione microscopi elettronici
• SE153001.0001 Manutenzione forni
• SE153004.0001 Taratura strumentazione di laboratorio
```

**Ma perde sul contenuto al secondo turno**: dice *"Aucun MMD correspondant… n'a ete trouve pour
RSSE"* subito dopo aver trovato `SE103003.0001` al turno precedente. Il vecchio modello e Sonnet
lo indicavano come il piu' pertinente, da richiedere.

**`Claude Sonnet 4.6` da' le risposte migliori** — sezioni, disponibilita', alternative, francese
impeccabile — ma scrive molto, e **mi ha dato un `FlowActionTimedOut`**. Inoltre non mette il
prefisso `MMD | ` e usa tabelle ed emoji: il formato andrebbe riscritto per lui.

**`GPT-5 Chat` e' da escludere**: perde la lingua su entrambi i turni.

## Attenzione sui timeout: non sono del modello

Ho avuto `FlowActionTimedOut` con Sonnet **e** con Chat Latest, ma **rilanciando lo stesso caso
subito dopo, Chat Latest ha risposto correttamente**. Sono i timeout del flusso gia' noti
(2494), non un difetto dei modelli nuovi. Da non confondere.

## Stato lasciato (verificato ricaricando le pagine)

| agente | modello | stato |
|---|---|---|
| `Copilot for finance and operations apps` (`679d8694…`) | **GPT Chat Latest** | pubblicato 30/10 |
| `RINA Procurement & MMD Assistant` (`dfd051a3…`) | **GPT Chat Latest** | pubblicato 30/10, ore 11:08 |

Gli agenti connessi (`b4a1f29d…` figlio, `RDA Creation Assistant`) non hanno selettore:
ereditano dal padre, quindi sono anche loro su GPT Chat Latest.

**Tutto l'ambiente e' allineato su un solo modello.** Nessun agente resta su GPT-5.5 Chat.

**Nota su `GPT Chat Latest`**: segue sempre l'ultimo modello disponibile, quindi **il motore puo'
cambiare senza preavviso**. Il piano del 21/10 lo sconsigliava proprio per questo su un agente
instabile. Oggi e' il migliore dei tre sulla forma: la scelta e' fra avere il formato giusto
adesso e sapere cosa gira sotto.
