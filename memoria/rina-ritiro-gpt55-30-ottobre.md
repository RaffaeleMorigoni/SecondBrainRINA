---
name: rina-ritiro-gpt55-30-ottobre
description: "Cambio fatto il 30/10: provati tre modelli, scelto GPT Chat Latest su entrambi gli agenti - e'l'unico col formato giusto, ma perde il filo al secondo turno"
metadata:
  type: project
---

**Fatto il 30/10/2026.** Entrambi gli agenti con selettore passati da `GPT-5.5 Chat` a
**`GPT-5 Chat`** e pubblicati: il padre `679d8694…` (10:19) e lo standalone `dfd051a3…` (10:22).
Gli agenti connessi non hanno selettore: ereditano dal padre.

Non era rinviabile: l'avviso diceva *"If you don't change the model, we'll switch to a newer one"*
— se non sceglievamo noi, sceglieva Microsoft.

## L'esito: regressione netta

Confronto con un "prima" rilevato **quaranta minuti prima**, stesse istruzioni e stesso flusso
(`plans/20261030_PRIMA_del_cambio_modello.md`):

- **francese rotto su entrambi i turni**: risponde **in italiano**, una riga sola, senza
  disponibilita' RSSE/PATR ne' alternative; al secondo turno **ripete identica** la risposta del
  primo. Prima era corretto in francese, due volte su due.
- **italiano**: `MMD | ` **ripetuto su ogni riga** (le istruzioni dicono "once at the very
  beginning") e **sparita tutta la prosa**.
- unica cosa migliorata: **l'elenco ora va uno per riga**, il difetto di formato che inseguivamo
  da settimane ([[rina-template-risposta-ricerca]]).

## Regola che ne esce

**Un "prima" vale solo se e' dello stesso giorno.** La baseline del 21/10 non era piu' usabile:
in mezzo avevamo cambiato filtro del flusso e due blocchi di istruzioni. Rifarla e' costato
mezz'ora e ha salvato il confronto — e ha anche fatto emergere che il caso francese a due turni
era **gia' intermittente prima** del cambio (una volta su due chiedeva la lingua), quindi quel
difetto non va attribuito al modello nuovo.

## Da fare

1. **riprovare la suite dopo qualche ora** (il comportamento si assesta dopo una pubblicazione);
2. se resta cosi', **provare `GPT Chat Latest` o `Claude Sonnet 4.6`**, uno alla volta, con la
   stessa coppia di casi;
3. **non toccare le istruzioni ora**: motore appena cambiato, una variabile alla volta.

Dettaglio in `plans/20261030_cambio_modello_GPT5.md`.


## Il confronto fra tre modelli (30/10, stessa mattina)

Provati tutti sul padre, stessi due casi, istruzioni e flusso invariati.

| | GPT-5.5 (vecchio) | GPT-5 Chat | Claude Sonnet 4.6 | **GPT Chat Latest** |
|---|---|---|---|---|
| prefisso `MMD \| ` | 1 volta | su ogni riga | assente | **1 volta** |
| elenco a capo | **no** | si' | si' (tabella) | **si'** |
| francese | ok | **italiano** | il migliore | ok |
| 2o turno FR | intermittente | ripete il 1o | il migliore | ok ma *"aucun MMD trouve"* |

**Scelto `GPT Chat Latest`** ed e' ora su **entrambi** gli agenti con selettore, `679d8694…` e
`dfd051a3…`; i connessi ereditano. **Nessun agente resta su GPT-5.5.** E' l'unico che mette il
prefisso una volta **e** manda l'elenco a capo: il formato di
[[rina-template-risposta-ricerca]] finalmente giusto. Difetto suo: al secondo turno dice che non
ha trovato nulla subito dopo aver trovato il codice.

**`Claude Sonnet 4.6` da' le risposte migliori** (sezioni, disponibilita', alternative, francese
impeccabile) ma e' prolisso, non mette il prefisso, usa tabelle ed emoji e ha dato un timeout:
per adottarlo andrebbe riscritto il formato. Tenerlo come candidato.

**Rischio noto di `GPT Chat Latest`**: segue sempre l'ultimo modello, quindi **il motore puo'
cambiare senza preavviso**. Quando qualcosa si rompera', verificare il modello prima di dare la
colpa a una nostra modifica.

**I `FlowActionTimedOut` non sono del modello**: ne sono arrivati con Sonnet e con Chat Latest,
ma rilanciando subito lo stesso caso Chat Latest ha risposto. Sono i timeout del flusso del 2494.


## 10/11/2026 — l'etichetta ora e' versionata: "GPT Chat Latest (Oct 2026)"

Il 30/10 il selettore diceva solo **GPT Chat Latest**; oggi dice **GPT Chat Latest (Oct 2026)**.
Microsoft ha reso visibile quale snapshot c'e' sotto. E' il rischio gia' annotato - l'opzione
segue l'ultimo modello - con la differenza che ora **si vede** quando cambia: se l'etichetta
cambia data, e' cambiato il motore.

**Conseguenza pratica scoperta oggi**: qualunque segnalazione **anteriore al 30/10** e' stata
prodotta su **GPT-5.5 Chat**, che non esiste piu'. Confrontarla con una prova di oggi significa
confrontare due agenti diversi. E' successo con la tabella dei casi Dafne/Alba
([[rina-casi-dafne-alba]]): due casi su sei, rigiocati, danno esito diverso.

**Regola**: prima di aprire un difetto da una segnalazione, controllare **se e' precedente al
cambio di modello**. La data di ritiro da tenere a mente e' il **30/10/2026**.
