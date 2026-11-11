# La prosa italiana dipende dalla lingua di interfaccia F&O — A/B del 30/10/2026

Test chiesto per isolare la variabile residua dopo **otto tentativi falliti** di governare la
lingua dal prompt (vedi `plans/backup/20261029_ROLLBACK_padre_copied_as_it_is.md`).

## Metodo

A/B **nello stesso giorno, a mezz'ora di distanza**, con una sola variabile cambiata.
Il 19/10 un confronto fra giorni diversi mi aveva portato a una diagnosi sbagliata: qui no.

| | |
|---|---|
| ambiente | F&O UAT, societa' **RSSE**, pagina *Dettagli prodotto rilasciato* |
| utenza | `C_110912@rina.org` |
| domanda | `Please give me the details of item number SE108002.0001` (identica nelle due prove) |
| sessione | Copilot aperto da zero in entrambi i casi |
| variabile | **solo** Opzioni utente > Preferenze > Lingua |

## Risultato

### Lingua utente `en-US` — risposta **tutta in inglese**

```
MMD | SE108002.0001 High-altitude mechanical systems maintenance
Additional details:
- Type: Service
- Category: Mainten
- Unit: QTY
- Price: 0
- Vendor: not specified
```

Prova: `prove_uat/20261030_interfaccia_EN_risposta_tutta_inglese.png`

### Lingua utente `it` — risposta **tutta in italiano**

```
MMD | SE108002.0001 High-altitude mechanical systems maintenance
- Tipo: servizio
- Categoria: Mainten
- Unita' di misura: QTY
- Prezzo registrato: 0
- Fornitore predefinito: non indicato
Questo articolo risulta disponibile nella societa' RSSE.
```

Prova: `prove_uat/20261030_interfaccia_IT_risposta_tutta_italiana.png`

## Cosa dimostra

**La lingua della risposta la decide la lingua di interfaccia F&O, non la lingua della domanda
e non le istruzioni.** La domanda e' la stessa, il modello e' lo stesso, le istruzioni sono le
stesse (padre a 7.859, figlio a 29.447): cambia solo il locale dell'utente, e la risposta si
ribalta per intero.

Il codice e la descrizione restano corretti in tutti e due i casi: cambia **solo la forma**.

**Spiega perche' otto tentativi non hanno spostato niente.** Non stavamo correggendo un modello
disobbediente: il contesto di sessione F&O porta il locale e vince sul prompt. Nel pannello di
test quel locale non esiste, ed e' per questo che li' l'inglese e' sempre uscito giusto.

**Nota sul 19/10.** La memoria `rina-lingua-interfaccia-fo` diceva che il locale *non* era la
causa, sulla base di prove del 19/10 fatte con interfaccia `it` che uscivano in inglese. Quel
quadro non regge piu': l'A/B di oggi e' sulla stessa giornata e sulla stessa domanda. Possibile
che il 19/10 la risposta fosse ancora instabile, o che le modifiche successive al flusso e alle
istruzioni abbiano reso il difetto sistematico. Quello che conta e' che **oggi e' riproducibile
a comando, in due direzioni**.

## Cosa farne

1. **Smettere di riscrivere regole sulla lingua.** Otto tentativi, zero spostamenti, e ora
   sappiamo perche'.
2. **Portarlo a Microsoft come limite della piattaforma**, con queste due prove affiancate: e'
   il materiale piu' solido che abbiamo prodotto sul tema.
3. **Dirlo a Emanuela e sulla segnalazione FOFI** nei termini giusti: chi prova con interfaccia
   italiana vedra' sempre la prosa italiana, qualunque lingua usi per chiedere. Non e' un
   difetto dell'agente che possiamo correggere dalle istruzioni, e non tocca il dato.
4. **Aggiornare il criterio dei test**: la lingua di interfaccia di chi prova va dichiarata in
   ogni segnalazione, altrimenti i casi non sono confrontabili.

## Stato lasciato

Lingua utente **riportata a `it`** e salvata. Nessuna altra modifica.

---

# RETTIFICA — la terza cella cambia la conclusione (30/10, stessa mattina)

Su obiezione di Raffaele (*"ma la lingua non e' determinata da come scrive?"*) ho chiuso la
casella mancante della matrice: **interfaccia `en-US` + domanda in italiano**.

```
Dammi i dettagli dell'articolo numero SE108002.0001

MMD | SE108002.0001 Manutenzioni impianti meccanici in quota
Dettagli disponibili:
- Tipo: service
- Categoria: Mainten
- Unita' di misura: QTY
- Prezzo: 0
- Legal entity: RSSE
```

Prova: `prove_uat/20261030_interfaccia_EN_domanda_IT_risposta_italiana.png`

**Risposta in italiano, con interfaccia inglese.** Cioe' qui la lingua della domanda **vince**
sul locale.

## La matrice completa

| | domanda IT | domanda EN |
|---|---|---|
| **UI `it`** | (atteso italiano, coerente) | **italiano** ❌ |
| **UI `en-US`** | **italiano** ✅ | **inglese** ✅ |

## Conclusione corretta

**"Comanda il locale" era troppo forte: e' sbagliato.** Su tre celle provate, due sono corrette.
La lingua della domanda viene rispettata, **tranne in un caso**: quando il locale e' italiano e
la domanda non lo e'. Li' l'italiano trascina tutto.

Formulazione giusta: **l'italiano ha la precedenza.** Basta un solo segnale italiano — il locale
*oppure* la domanda — perche' la risposta esca in italiano. Per avere una risposta inglese devono
essere inglesi **tutti e due**.

## Il dato invece e' sempre giusto

La **descrizione** segue sempre la lingua della domanda, in entrambi i locali:
- domanda EN → `High-altitude mechanical systems maintenance`
- domanda IT → `Manutenzioni impianti meccanici in quota`

Quindi la catena a monte (flusso e traduzioni, `ProductTranslations`) **conosce la lingua della
domanda e la usa correttamente**. A lasciarsi trascinare dall'italiano e' solo la **prosa di
contorno** che compone chi scrive la risposta finale. Il difetto e' molto piu' stretto di quanto
sembrasse stamattina.

Si vede anche nei residui misti: `Tipo: service`, `Legal entity: RSSE` dentro una risposta
italiana. Non c'e' un template: le etichette vengono improvvisate ogni volta, e pescano ora da
una lingua ora dall'altra.

## Cosa cambia per il cliente

Il caso rotto e' **solo** quello dei tester italiani che provano in inglese — cioe' esattamente
quello della segnalazione FOFI. Un utente portoghese o francese, che ha il suo locale e scrive
nella sua lingua, **non e' toccato**: ha un solo segnale non italiano e nessun italiano che lo
sovrascriva. Questo va detto, perche' stamattina stavo per descrivere il difetto come generale.
