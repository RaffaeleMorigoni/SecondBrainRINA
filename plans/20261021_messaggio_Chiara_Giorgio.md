# Messaggio per Chiara e Giorgio — 21/10/2026

Ciao a entrambe/i, torno dalle ferie e mi riallineo. Riassumo cosa ho fatto stamattina e tre cose
su cui dobbiamo metterci d'accordo prima di pestarci i piedi.

**Cosa ho chiuso oggi**

- **2419** (francese su RSPA): risolto e verificato, l'ho messo a Done. La causa era nel flusso:
  la catena delle traduzioni che avevo costruito il 16/10 copriva solo gli articoli *disponibili*
  in societa', mentre il ramo "da richiedere" leggeva il campo italiano. Ho aggiunto una catena
  dedicata a quel ramo. Verificato sull'output reale: ora esce
  `Services de surveillance non armée` dove prima usciva `Servizi di sorveglianza non armata`.
  La query gia' esistente non l'ho toccata, per non appesantire il flusso lato tempi.

- **2418** (francese su RSSE): **l'ho lasciato aperto**, perche' non e' un problema di traduzione.
  L'ho riprodotto e sono andato a cercare la run nel flusso: **non esiste**. L'agente ha risposto
  **senza chiamare Search Items**, prendendo la descrizione da un'altra fonte in italiano e
  aggiungendo le traduzioni fra parentesi di suo. Riformulando la stessa domanda in modo piu'
  esplicitamente "di catalogo", il flusso viene chiamato e la risposta esce tutta in francese.
  Quindi e' instradamento, non lingua: va lavorato sulle regole dell'orchestratore.

  Criterio utile per i prossimi casi: **se vedete "descrizione italiana (traduzione fra
  parentesi)", quella risposta non viene dal flusso.** Quando l'elenco arriva dal flusso le
  descrizioni sono gia' nella lingua giusta, senza parentesi.

**Sul documento CAPEX/OPEX — chiarisco l'equivoco, perche' avevamo ragione tutti e due**

Chiara, il file che cercavi c'era davvero, ma in un altro punto: `Summary capex e opex 1.pdf` e'
nella Knowledge del bot **RINA Procurement & MMD Assistant**, caricato da me il 9 ottobre, stato
Ready. Tu hai guardato nel **padre**, dove effettivamente non c'era.

Pero' **hai fatto la cosa giusta a caricarlo li'**: ho verificato che la configurazione che conta
oggi (knowledge e strumenti) sta tutta sul padre, assegnata al nodo child. Il bot standalone dove
l'avevo messo io risulta di fatto non piu' esercitato — ultima attivita' reale il 15/10. Quindi
senza la tua copia il documento oggi non sarebbe stato disponibile.

Resta una copia orfana sul bot standalone: propongo di rimuoverla, perche' e' un'esca per il
prossimo che va a controllare. Ma non la tocco senza un vostro ok.

**Attenzione: le istruzioni dell'orchestratore sono a 7.988 su 8.000 caratteri**

Restano **12 caratteri**. Oltre il limite Copilot Studio **tronca in silenzio**, senza nessun
avviso, e l'ultimo blocco e' `ONE ANSWER PER TURN`, che contiene sia la regola anti-doppio-
messaggio (difetto 2333) sia quella che fa tradurre le risposte di ricerca. Se si tronca, si
riaprono insieme 2333 e i quattro task lingua.

Ho preparato una riscrittura che libera **704 caratteri** senza togliere nessuna regola (solo una
riga duplicata e quattro blocchi resi piu' densi): porta il campo da 7.989 a 7.285. Prima di
applicarla volevo sentirvi, perche' il padre l'avete pubblicato ieri e non voglio sovrascrivere
modifiche vostre in corso.

**Da concordare**

1. Chi tocca l'orchestratore e quando: oggi ci abbiamo lavorato in piu' persone e quel campo non
   ha cronologia, quindi una sovrascrittura si perde senza lasciare traccia.
2. Il 2418 (instradamento) lo prendo io o lo vedete voi?
3. Ok a rimuovere la copia orfana del PDF sul bot standalone?

Grazie,
Raffaele
