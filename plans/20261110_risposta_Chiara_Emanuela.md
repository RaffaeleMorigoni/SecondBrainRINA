# Bozza di risposta a Chiara ed Emanuela — 10/11/2026

*(da rileggere e inviare tu — non ho mandato niente)*

---

Ciao Chiara, ciao Emanuela,

grazie della tabella: e' molto utile averli tutti insieme. Ho verificato i codici in anagrafica e
ho provato a riprodurre i casi. Vi riporto cosa ho trovato, perche' **i sei casi non sono tutti
la stessa cosa** e secondo me conviene trattarli separatamente.

**Li ho rigiocati tutti e sei oggi, sul modello attuale. Quattro non si riproducono piu'.**

| caso | esito oggi |
|---|---|
| Ferro per fonderia | **corretto**: *"non risulta attualmente rilasciato nella societa' RCSM e deve essere richiesto"* |
| Project Planning | **corretto**: risponde che non trova articoli pertinenti |
| Training | **corretto**: `SE702502.0001` e' fra le proposte |
| Resistenza per impianti pilota | **migliorato**: propone `SE153006.0001` *Manutenzione impianti pilota macchine di testing* |
| Bottiglie in plastica e vetro | invariato, ma non e' un bug (vedi sotto) |
| **Supporto tecnico onsite** | era l'unico difetto vero: **corretto oggi** |

Qualche dettaglio sui piu' significativi.

**Ferro per fonderia.** L'ho provato **in RCSM** e l'agente risponde esattamente come vi
aspettavate: da' il codice e precisa che in quella societa' non e' rilasciato e va richiesto.
Segnalo pero' una cosa che ho scoperto guardando l'anagrafica di RCSM: **li' esistono solo codici
legacy** (CSMFon1301 Barrotti di grafite, CSMFon1303 Forni, CSMFon1306 Ricambi per forni…),
**nessun codice MMD nuovo**. E' la stessa situazione gia' nota per RSPA. Quindi per i tester di
quella societa' conviene saperlo: non e' l'agente che non trova, e' che li' non c'e' ancora
niente da trovare nel formato nuovo.

**Resistenza per impianti pilota.** Qui l'agente oggi propone `SE153006.0001` —
**"Manutenzione impianti pilota macchine di testing"** — e precisa che non e' rilasciato in RSSE
ma disponibile in PATR. Rispetto all'atteso `SE153002.0001` (*manutenzione impianti di
laboratorio*) e' addirittura piu' specifico. **Dafne e Alba avevano ragione sull'impostazione**:
il codice giusto sta nella famiglia della manutenzione impianti, non fra i beni.

**Training.** `SE702502.0001` oggi c'e', al terzo posto. Aggiungo che sono riuscito a ritrovare
le sessioni originali di ottobre: la domanda (in francese) era stata posta tre volte, e in **due
delle tre** l'agente aveva gia' proposto quel codice. Non e' una contestazione della
segnalazione: e' il sintomo del problema che spiego piu' sotto.

**Bottiglie in plastica e vetro: questo non e' un errore di ricerca.** L'agente risponde
`MA101019.0001`, che in anagrafica e' **"Fornitura materiali-prodotti in vetro"**: ha agganciato
la parola *vetro* della richiesta, ed e' letteralmente corretto. Il codice atteso
`MA101037.0001` e' **"Fornitura materiali-prodotti di consumo per laboratori"**, cioe' la
categoria **d'uso**. Nessun articolo si chiama "bottiglie": per arrivare al codice atteso bisogna
sapere che in RINA le bottiglie da laboratorio si comprano come materiale di consumo — una
conoscenza che oggi l'agente non ha da nessuna parte. Se ci confermate che la regola e' questa,
possiamo scrivergliela.

**Supporto tecnico onsite: era l'unico difetto vero, e oggi l'ho corretto.**

Ho verificato in anagrafica che **nessun articolo contiene "onsite"**, quindi l'osservazione era
giusta. Sono andato a vedere nel dettaglio cosa succedeva e la causa era diversa da quella che
mi aspettavo: **i codici venivano davvero dalla ricerca**, l'agente non se li inventava. Cercando
le parole *supporto* e *tecnico* l'anagrafica restituisce per forza cose come
*Laser scanning per supporto alla progettazione* o *Monitoraggio geotecnico* — e l'agente aveva
l'istruzione di elencare **tutti** gli articoli trovati, quindi li elencava.

Ho cambiato quell'istruzione: ora deve riportare **solo gli articoli che rispondono davvero alla
richiesta**, e se nessuno e' pertinente deve dirlo. Pubblicato oggi pomeriggio. Il risultato:

> *MMD | Nessun articolo trovato corrisponde in modo sufficientemente preciso a
> «Supporto tecnico onsite».*

Ho verificato che non si sia rotto nulla: *"servizio di manutenzione per le attrezzature di
laboratorio"* continua a rispondere correttamente con `SE153005.0001`.

**Una cosa su cui vi chiedo di tenere gli occhi aperti.** Questa modifica rende l'agente piu'
selettivo, e si vede: la stessa domanda sulla manutenzione, che a ottobre restituiva **sei**
codici fra cui scegliere, oggi ne restituisce **uno solo** — centrato, ma uno.

Per una richiesta precisa va benissimo. Per una richiesta generica, dove avere una rosa di
alternative aiuta a scegliere, **potrebbe essere diventato troppo stretto**. Non ho elementi per
dire se sia un problema o no, e preferisco non calibrarlo a sensazione.

Quindi: **se nei prossimi giorni vi capita di pensare "mi da' un codice solo e non e' quello che
cercavo", segnalatemelo** — e' questa la causa, non un difetto nuovo, e si corregge in pochi
minuti allargando la selezione. E' l'unico punto della modifica su cui ho bisogno del vostro
riscontro sul campo: dall'interno non riesco a stabilire quale sia il compromesso giusto fra
"pochi codici precisi" e "una rosa fra cui scegliere".

**Perche' quattro casi su sei sono cambiati: ci sono due motivi, e vale la pena conoscerli.**

Il primo: **il 30 ottobre Microsoft ha ritirato GPT-5.5 Chat**, il modello su cui girava
l'agente, e l'ho dovuto spostare su **GPT Chat Latest** (la console oggi lo indica come
*Oct 2026*). I vostri casi sono **anteriori al 9 ottobre** — l'ho verificato perche' non sono
piu' nel monitoraggio, che conserva 29 giorni — quindi sono stati prodotti con un modello che
**non esiste piu'**. In pratica la tabella fotografa un agente diverso da quello in esercizio.
Non e' un rimprovero a nessuno: il cambio e' stato imposto da Microsoft e l'abbiamo fatto nei
tempi.

Il secondo, piu' fastidioso: **oggi l'agente non da' sempre la stessa risposta alla stessa
domanda**. Le parole chiave con cui interroga l'anagrafica cambiano a ogni esecuzione, e con
quelle cambiano i risultati. E' il motivo per cui su *onsite* escono codici diversi ogni volta.
Ci sto lavorando ed e' la prima cosa che voglio chiudere, perche' finche' resta cosi' **ogni
altra correzione e' difficile da verificare**.

**Cosa mi servirebbe da voi, per essere efficace**

1. **Ripetere ogni caso due o tre volte** prima di metterlo in tabella, e annotare se la risposta
   cambia. Non e' un dubbio sulle segnalazioni: e' che con una sola prova rischiamo di inseguire
   qualcosa che si sposta da solo. Oggi, rigiocandoli, **quattro casi su sei** hanno dato un
   risultato diverso da quello in tabella.
2. **La societa' (legal entity)** indicata insieme al caso, d'ora in avanti. Con *Ferro per
   fonderia* me la sono cavata perche' l'attesa nominava RCSM e ho potuto provare li', ma negli
   altri casi ho dovuto tirare a indovinare: la ricerca lavora sulla societa' della sessione, e
   lo stesso codice puo' essere disponibile in una e da richiedere in un'altra.
3. **Il testo esatto digitato**, non il riassunto. La colonna "Richiesta dell'utente" e' sintetica
   (*"Project Planning"*, *"Training OPEX/diretti"*) e la ricerca dipende molto dalle parole
   usate: con parole diverse vengono fuori codici diversi.
4. Per *Resistenza*, la conferma sulla convenzione di cui sopra.
5. **Un riscontro sulla selettivita'**: se vi sembra che ora l'agente dia troppi pochi codici,
   ditemelo (vedi la nota sopra). E' l'unica cosa che non riesco a valutare dall'interno.

Un'ultima cosa utile per il futuro: **il monitoraggio conserva le conversazioni solo 29 giorni**.
Cinque dei sei casi erano gia' fuori finestra e non sono piu' recuperabili. Se le segnalazioni
arrivano entro il mese riesco a risalire alla sessione originale e a vedere esattamente cosa ha
fatto l'agente — il che fa una grande differenza.

Grazie ancora, e scusate la lunghezza.
Raffaele
