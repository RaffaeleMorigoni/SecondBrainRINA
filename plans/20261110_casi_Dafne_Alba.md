# Tabella Chiara — codici restituiti vs attesi (casi Dafne e Alba)

Ricevuta il 10/11 da Chiara, inoltrata a Emanuela. File originale:
`Downloads/20261110_codici_MMD_risultato_vs_atteso (1).xlsx` — **non modificato**.

| # | richiesta | agente | atteso dall'utente |
|---|---|---|---|
| 1 | Bottiglie in plastica e vetro | `MA101019.0001` | `MA101037.0001` |
| 2 | Ferro per fonderia | `MA101017.0001` | *"lo segnala come non disponibile in RCSM"* |
| 3 | Supporto tecnico onsite | `PS104002.0001`, `PS104003.0001`, `PS104004.0001`, `PS104005.0001` | *"Non esiste articolo Supporto Tecnico Onsite o Site"* |
| 4 | Project Planning | `SE703502.0001`, `PS101002.0080` | *"Non esiste un articolo pertinente"* |
| 5 | Training OPEX/diretti | `PS101003.0018`, `PS201019.0001`, `SE356004.0001` | `SE702502.0001` **assente tra le proposte** |
| 6 | Resistenza per impianti pilota | `ME201003.0001` | `SE153002.0001` |

## Non sono sei casi dello stesso tipo: sono due difetti diversi

**A — L'agente propone codici dove l'utente dice che non ce ne sono** (casi 2, 3, 4).
Tre su sei. In questi l'atteso **non e' un codice**, e' un giudizio: *"non esiste"*,
*"non disponibile in RCSM"*. Se e' vero che non esiste nulla di pertinente, allora l'agente
**sta proponendo codici che non dovrebbe**, ed e' lo stesso difetto misurato il 30/10
([[rina-codici-fuori-dalla-ricerca]]): codici che non vengono dalla ricerca ma dalla Knowledge
o dalla memoria del modello, con un formato impeccabile che li fa sembrare giusti.

**B — L'agente non trova il codice giusto** (casi 1, 5, 6), o sbaglia societa' (caso 2).
Qui l'atteso e' un codice preciso che esiste: va verificato **perche'** la ricerca non l'ha
trovato — parole chiave, campo interrogato, societa'.

## Il caso 2 e' anche un tema di societa'

*"non disponibile in RCSM"*: **RCSM e' un'altra legal entity**. Da verificare in quale societa'
girava la sessione e se `MA101017.0001` sia rilasciato in RCSM. E' il tema gia' noto della
company della sessione ([[rina-company-sessione-dataareaid]]) e delle due liste
disponibile / da richiedere ([[rina-search-items-etichette-disponibilita]]).

## Il caso 6 va guardato con attenzione, perche' l'atteso e' strano

Una **resistenza per impianti pilota** e' un **componente fisico**. L'atteso `SE153002.0001` e'
invece — stando a quanto letto il 30/10 in F&O — **"Manutenzione installazioni di laboratorio"**,
cioe' un **servizio di manutenzione**. L'agente ha risposto `ME201003.0001`, che per prefisso
sembra un materiale.

**Non sto dicendo che l'utente abbia torto**: puo' esserci una convenzione RINA per cui quel
ricambio si acquista sotto la manutenzione di laboratorio. Ma e' il primo controllo da fare,
perche' se l'atteso e' sbagliato il caso non e' un difetto dell'agente.

## Piano di verifica (serve F&O)

Per ciascuno dei sei casi, nell'ordine:

1. **I codici esistono?** Controllare in anagrafica `EcoResProductDetailsExtendedGrid` sia i
   codici proposti sia gli attesi, con descrizione e societa' di rilascio.
2. **In quale societa' girava la sessione?** Senza quel dato i casi 2 e 3 non sono giudicabili.
3. **Riprodurre le sei domande** e, per ognuna, **confrontare i codici in risposta con gli output
   delle `Lista prodotti`** in Power Automate — la regola del 30/10. Serve a distinguere
   "la ricerca ha trovato roba non pertinente" da "l'agente ha inventato".
4. Leggere `Filtra parole` di ogni run: le keyword cambiano a ogni esecuzione
   ([[rina-keyword-non-deterministiche]]), quindi un singolo esito non basta.

## Cosa chiederei a Chiara prima di rispondere a Emanuela

- **In che societa'** stavano lavorando Dafne e Alba quando hanno provato.
- **Le domande esatte** poste all'agente: la colonna "Richiesta dell'utente" e' una sintesi
  (*"Project Planning"*, *"Training OPEX/diretti"*), non il testo digitato. La ricerca dipende
  dalle parole usate, quindi senza il testo originale non si riproduce il caso.
- Per il caso 6, **conferma che `SE153002.0001` sia davvero il codice giusto** per una resistenza.

---

# Verifica in anagrafica F&O (RSSE, 10/11)

## I codici: cosa sono davvero

| codice | descrizione in F&O | |
|---|---|---|
| `MA101019.0001` | **Fornitura materiali-prodotti in vetro** | proposto dall'agente, caso 1 |
| `MA101037.0001` | **Fornitura materiali-prodotti di consumo per laboratori** | atteso, caso 1 |
| `ME201003.0001` | **Strumenti da laboratorio e standard di riferimento** | proposto dall'agente, caso 6 |
| `SE153002.0001` | **Manutenzione impianti di laboratorio** | atteso, caso 6 |

Tutti e quattro **esistono e sono rilasciati in RSSE**.

## Ricerche per parola

| cercato | risultato in RSSE |
|---|---|
| `*bottigl*` | **nessun articolo** |
| `*onsite*` | **nessun articolo** |
| `*planning*` | **solo due codici legacy**: `PrENGdes…1404` (Planning, Supervision and Control) e `PrENGtechass1505` (Project control and planning) — **nessun MMD nuovo** |

## Cosa ne esce, caso per caso

**Caso 1 — Bottiglie in plastica e vetro: non e' un difetto di ricerca.**
L'agente ha risposto *"materiali-prodotti **in vetro**"*: ha agganciato la parola **vetro** della
domanda, ed e' letteralmente corretto. L'utente voleva invece *"materiali di consumo **per
laboratori**"*, cioe' la categoria **d'uso**. Nessun articolo si chiama "bottiglie": la
corrispondenza richiesta e' semantica, non testuale. **La ricerca per parole ha funzionato fin
troppo bene.**

**Caso 3 — Supporto tecnico onsite: l'utente ha ragione.**
Confermato: **nessun articolo contiene "onsite"**. L'agente ne ha proposti **quattro**
(`PS104002-005`). Qui il difetto e' reale ed e' quello grave del 30/10: **propone codici dove non
dovrebbe** ([[rina-codici-fuori-dalla-ricerca]]).

**Caso 4 — Project Planning: l'utente ha ragione, ma con una precisazione che conta.**
Esistono due articoli pertinenti, ma sono **solo legacy** (`PrENGdes…`, `PrENGtechass…`):
**nessun MMD nuovo** copre il project planning. L'agente ha proposto due codici nuovi
(`SE703502.0001`, `PS101002.0080`) che non c'entrano. E' insieme un difetto dell'agente **e** un
buco della tassonomia nuova ([[rina-tassonomia-nuova-senza-descrizioni]]).

**Caso 6 — Resistenza per impianti pilota: e' una convenzione, non un difetto.**
L'agente propone un **bene** (*strumenti da laboratorio*), l'utente si aspetta un **servizio**
(*manutenzione impianti di laboratorio*). Entrambi riferiti al laboratorio, entrambi esistenti.
Chi ha ragione dipende da **come RINA acquista i ricambi**: dentro un contratto di manutenzione,
oppure come bene. **L'agente non puo' indovinare una convenzione che non e' scritta da nessuna
parte** - ne' nelle istruzioni ne' nella Knowledge.

## Non ancora verificato

- `MA101017.0001` (caso 2) e **la disponibilita' in RCSM**: serve sapere in che societa'
  lavoravano Dafne e Alba.
- `PS104002-005`, `SE703502.0001`, `PS101002.0080` e i tre codici del caso 5: cosa sono.
- `SE702502.0001` (atteso caso 5): se esiste e perche' non e' stato proposto.
- **La provenienza dei codici**: riprodurre le sei domande e confrontare le risposte con gli
  output delle `Lista prodotti` in Power Automate, per distinguere *"la ricerca ha restituito
  roba non pertinente"* da *"l'agente ha inventato"*. Senza le domande esatte di Dafne e Alba
  pero' il caso non e' riproducibile fedelmente.

## Come lo direi a Emanuela

Dei sei casi, **due sono difetti veri dell'agente** (3 e 4: propone codici dove non ce ne sono),
**uno e' una convenzione aziendale da chiarire** (6), **uno e' un disallineamento semantico**
(1: l'agente prende la parola, l'utente pensa alla categoria d'uso), e **due restano da
verificare** (2 e 5).

E una cosa utile per loro: **il caso 4 dice che la tassonomia nuova ha un buco**. Non e' colpa
dell'agente se per "project planning" esistono solo codici vecchi.

---

# Ricerca nelle sessioni e repliche (10/11)

## Le sessioni originali: solo una delle sei e' recuperabile

Cercate nelle **205 sessioni** scaricate (9/10 - 6/11, orchestratore) per parole e per **codici
della tabella**. I test passano tutti da F&O, quindi dall'orchestratore; l'agente standalone ha
**zero sessioni** in 30 giorni, quindi e' escluso.

**Cinque casi su sei non ci sono**: le segnalazioni sono **anteriori al 9 ottobre**, fuori dalla
retention di 29 giorni. Non sono piu' recuperabili.

## Caso 5 (Training) — trovato, e la tabella e' imprecisa

Tre sessioni con la **stessa identica domanda**, in **francese**:

> *"Bonjour je cherche un code MMD pour la formation du personnel. Pouvez-vous m'aider?"*

| quando | codici proposti | `SE702502` c'era? |
|---|---|---|
| 15/10 17:48 | SE356004.0001, **SE702502.0001**, PS201019.0001, PS101003.0011, PS101003.0026 | **si', al secondo posto** |
| 16/10 10:50 | SE356004.0001, **SE702502.0001**, PS201019.0001, PS101003.0011 | **si'** |
| 16/10 13:01 | SE356004.0001/.0002, SE356001, SE356002, SE356003, SE356005, SE356006 | **no** |

**`SE702502.0001` era proposto in due esecuzioni su tre**, al secondo posto. La segnalazione
*"assente tra le proposte"* e' vera **solo per una** delle tre.

E si vede un'altra cosa: **tre risposte completamente diverse alla stessa domanda**. La terza
propone l'intera famiglia dei corsi (SE3560xx) che le prime due ignorano.
Inoltre **due risposte su tre sono in italiano** a una domanda francese.

## Caso 3 (Supporto tecnico onsite) — riprodotto, con codici diversi

Riproposto oggi in F&O, RSSE:

> *MMD | Per "Supporto tecnico onsite" risultano questi articoli disponibili:*
> - `PS101003.0007` — TECNICO MANUTENTORE ESTINTORI
> - `PS101003.0017` — Tecnico collaboratore CND e ANSF
> - `PS201012.0004` — Laser scanning per supporto alla progettazione
> - `PS652005.0001` — Monitoraggio geotecnico

**Il difetto e' confermato**: nessun articolo contiene "onsite", e l'agente ne propone comunque
quattro, per giunta **palesemente non pertinenti** (estintori, monitoraggio geotecnico).

Ma **i codici sono diversi da quelli segnalati** (`PS104002-005`): anche qui la risposta cambia
a ogni esecuzione.

## Caso 4 (Project Planning) — NON si riproduce

> *MMD | **Project Planning** — Non risultano informazioni disponibili nei contenuti aziendali
> ricercabili per "Project Planning".*

Oggi l'agente risponde **esattamente come l'utente si aspettava**. Il caso o e' gia' rientrato,
o e' intermittente come gli altri.

## Il filo che lega tutto

Delle tre situazioni verificabili, **due mostrano la stessa cosa**: la risposta alla stessa
domanda **cambia a ogni esecuzione**. Codici diversi, liste diverse, a volte lingua diversa.
E' lo stesso difetto misurato il 30/10 ([[rina-keyword-non-deterministiche]]).

**Conseguenza pratica, da dire a Emanuela**: una tabella "risultato vs atteso" costruita su
**una sola esecuzione per caso** non e' una base affidabile. Due dei sei casi, rifatti oggi,
danno un risultato diverso da quello segnalato — uno in meglio, uno con altri codici sbagliati.

Prove: `prove_uat/20261110_caso3_onsite_riprodotto_altri_codici.png`,
`prove_uat/20261110_caso4_planning_non_riprodotto.png`.

---

# TUTTI E SEI rigiocati sul modello nuovo (10/11, GPT Chat Latest Oct 2026)

| # | caso | esito oggi | |
|---|---|---|---|
| 1 | Bottiglie in plastica e vetro | `MA101019.0001` — Fornitura materiali-prodotti **in vetro** | **invariato** |
| 2 | Ferro per fonderia *(in RCSM)* | `MA101017.0001` — *"non risulta attualmente rilasciato nella societa' RCSM e deve essere richiesto"* | **corretto** |
| 3 | Supporto tecnico onsite | `PS101003.0007`, `PS101003.0017`, `PS201012.0004`, `PS652005.0001` | **difetto, codici diversi** |
| 4 | Project Planning | *"Non risultano informazioni disponibili…"* | **corretto** |
| 5 | Training OPEX diretti | `SE356004.0001`, `PS201019.0001`, **`SE702502.0001`** | **corretto** |
| 6 | Resistenza per impianti pilota | `SE153006.0001` — Manutenzione impianti pilota macchine di testing — *"non rilasciato in RSSE, disponibile in PATR"* | **migliorato** |

## Quattro casi su sei oggi non si riproducono piu'

**Caso 2 — risolto, ed esattamente come chiesto.** Provato **in RCSM**, la societa' dell'attesa:
l'agente da' il codice e aggiunge *"non risulta attualmente rilasciato nella societa' RCSM e deve
essere richiesto"*. E' letteralmente il comportamento che l'utente si aspettava.

**Caso 4 — risolto.** Risponde che non trova nulla, come atteso.

**Caso 5 — risolto.** `SE702502.0001` **e' fra le proposte**, al terzo posto. La segnalazione
*"assente"* non si riproduce (ed era gia' presente in 2 delle 3 sessioni di ottobre).

**Caso 6 — migliorato oltre l'atteso.** L'utente si aspettava `SE153002.0001` (*manutenzione
impianti di laboratorio*); l'agente propone `SE153006.0001` — **"Manutenzione impianti pilota
macchine di testing"**, che e' piu' specifico e piu' calzante con "impianti pilota". E indica la
disponibilita' per societa'. **La famiglia e' quella che l'utente indicava (SE153xxx,
manutenzione impianti), quindi aveva ragione sull'impostazione.**

## Cosa resta davvero aperto

**Solo il caso 3 (Supporto tecnico onsite)**, ed e' un difetto reale: nessun articolo contiene
"onsite", e l'agente ne propone comunque quattro, non pertinenti (estintori, monitoraggio
geotecnico). **Sopravvive al cambio di modello**, cambiano solo i codici.

**Il caso 1 resta invariato ma non e' un bug**: l'agente aggancia *vetro* ed e' letteralmente
corretto; per arrivare a "materiali di consumo per laboratori" servirebbe una conoscenza di
dominio che oggi non ha.

## Una scoperta collaterale su RCSM

In **RCSM l'anagrafica contiene solo codici legacy** (`CSMFon1301` Barrotti di grafite,
`CSMFon1303` Forni, `CSMFon1306` Ricambi per forni, `CSMGar…`, `CSMHw…`): **nessun MMD nuovo**.
E' la stessa situazione gia' nota per RSPA ([[rina-rspa-anagrafica-mmd-vuota]]). Va tenuto
presente quando i tester di quella societa' segnalano "non trova": li' **non c'e' ancora niente
da trovare** nel formato nuovo.

Prove: `prove_uat/20261110_caso1_*`, `_caso2_RCSM_*`, `_caso3_*`, `_caso4_*`, `_caso5_*`,
`_caso6_*`.
