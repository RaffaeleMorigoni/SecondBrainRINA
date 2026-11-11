# ROLLBACK — istruzioni del padre, "copied as it is" (29/10/2026)

Orchestratore *Copilot for finance and operations apps*, blocco `PROCUREMENT RESULTS`
(in coda a `ONE ANSWER PER TURN`). Istruzioni a **7.859 / 8.000** prima della modifica.

## Testo ORIGINALE

```
PROCUREMENT RESULTS: that agent never writes to the user. Your reply for the turn is its resolvedResult, sent once and copied as it is, with the MMD label, all items with code and description, and no version of your own before or after it. Write it in the user's language, never reformat it or drop its MMD label. Each item: code plus description, never field: value pairs. The prefix must be exactly MMD | : if it differs, fix it to MMD | .
```

## Testo NUOVO

```
PROCUREMENT RESULTS: that agent never writes to the user. Your reply for the turn is its resolvedResult, sent once, with the MMD label, all items with code and description, and no version of your own before or after it. Write it in the user's language, never reformat it or drop its MMD label. Also when you answer from the tool results yourself, write every sentence in the user's language. Each item: code plus description, never field: value pairs. The prefix must be exactly MMD | : if it differs, fix it to MMD | .
```

## Perche'

**Due ordini in conflitto nella stessa frase**: *"copied as it is"* e *"Write it in the user's
language"*. Quando il contenuto arriva gia' in italiano, vince il primo.

**E il blocco parla di un `resolvedResult` che in quel percorso non esiste.** La traccia del
pannello di test (29/10) mostra che il padre chiama **direttamente** lo strumento
`RINA - Search Items` — `Execution summary` con i suoi output, nessun `resolvedResult`, nessun
`answered`. Il figlio non entra. Quindi "copia com'e'" finisce applicato ai **dati dello
strumento**, che sono italiani.

La seconda frase copre esplicitamente il caso reale: quando e' il padre a comporre la risposta
dai risultati dello strumento.

**Il divieto di aggiungere roba propria resta**: lo tiene *"and no version of your own before or
after it"*, che non e' stato toccato.

Conteggio: da 7.859 a **7.938 / 8.000**. Restano 62 caratteri liberi — attenzione, il limite
tronca in silenzio l'ultimo blocco.

---

## ESITO: fallito, RIPRISTINATO alle 17:20

Pubblicato, provato in F&O (RSSE, sessione nuova, *"Please give me the details of item number
SE108002.0001"*): risposta **ancora in italiano** — `Dettagli:`, `Tipo: servizio`,
`Unita' di misura`, `Societa'` — con la sola descrizione in inglese.

Istruzioni riportate a **7.859** caratteri e ripubblicate. Nessuna modifica lasciata in piedi.

**Ottavo tentativo fallito.** E questa volta il bersaglio era giusto — il padre, che e' davvero
chi compone la risposta — e l'intervento era del tipo che oggi ha funzionato due volte (togliere
l'ordine che contraddice, non aggiungere un divieto). Non ha spostato nulla lo stesso.

## Cosa ne concludo

Il difetto **non e' governabile dalle istruzioni**, ne' del figlio ne' del padre. E' coerente con
quanto gia' documentato: in F&O il formato non segue le istruzioni e si comporta diversamente dal
pannello di test, dove **la stessa domanda esce in inglese corretto**.

Nel pannello di test, a parita' di istruzioni e di modello, l'inglese funziona. In F&O no.
La variabile che resta e' il **contesto di sessione F&O**, che non passa dal prompt.

**Da smettere**: riscrivere regole sulla lingua. Otto tentativi, nessuno ha spostato l'inglese.
**Da fare**: portarlo come limite della piattaforma, con le prove dei due canali a confronto.
