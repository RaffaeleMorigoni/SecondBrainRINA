---
name: rina-test-regressione-manutenzione
description: Caso di test canonico per l'agente RINA - manutenzione attrezzature, da rieseguire dopo ogni modifica alle istruzioni
metadata:
  type: project
---

**Domanda:** "Che codice devo usare per la manutenzione di attrezzature?"
(societa di test RSSE, prima domanda di una sessione pulita)

**Risposta attesa** (baseline verificata il 2026-09-29 alle 11:54, istruzioni a 20162 caratteri):
- codice principale **SE152002.0001 - Manutenzione attrezzature tecnologiche**, mai SE105001.0001
- **due sezioni distinte**: i codici disponibili nella societa dell'utente e, sotto un titolo
  separato, quelli presenti in Dynamics ma da richiedere (tipicamente **SE152001.0001**, etichettato
  solo come "non rilasciato nella tua societa": su richiesta di Emanuela le altre LE non vanno mai nominate)
- **nessuna categoria legacy** (Prbpro220, Npimmo300 e simili non devono mai comparire)
- tempo di risposta intorno ai **40-45 secondi**

**Why:** questo singolo caso intercetta tutti e tre i difetti segnalati dai tester nella Wave del
2026-09-29 ed e sensibile alle regressioni. Due tentativi di aggiungere la riga della tassonomia
hanno fatto perdere le due sezioni e raddoppiare i tempi a 85s, senza alcun errore visibile altrove.

**How to apply:** rieseguirlo **sempre in una sessione di test nuova** dopo ogni modifica alle
istruzioni del child agent, e confrontare i quattro punti sopra. Se la risposta torna a essere un
paragrafo unico o i tempi superano il minuto, la modifica va annullata.
Vedi [[rina-tassonomia-nuova-senza-descrizioni]] e [[rina-search-items-etichette-disponibilita]].

**Esito Wave 2 (test del 28/09 di Roberto Mantese, rieseguiti il 29/09 dopo i fix):**

| Richiesta | Atteso | Prima | Dopo |
|---|---|---|---|
| reagenti chimici | MA101021.0001 | KO | **OK** |
| prestazione professionale ISO 9001 | PS101002.000x | KO | **OK** (PS101002.0002) |
| sviluppo soluzioni informatiche a CAPEX | SE302501.0001 o SE707005.0001 | KO | **OK** (entrambi) |
| "laboratorio" (parola singola) | - | KO | chiede di specificare, nessun tool chiamato |

Il pattern dei KO era sempre lo stesso: l'agente trovava la **categoria** giusta (es. MA101021) ma
non il **materiale** (MA101021.0001) e rispondeva "non trovato" elencando categorie, anche legacy.
Risolto dall'ampliamento della finestra di ricerca. Il quarto caso non e un errore: "laboratorio" da
solo e ambiguo e l'agente chiede di precisare, ma il comportamento va validato col tester.

**Baseline aggiornata alle 13:01 del 29/09** (istruzioni a 20648 caratteri): la risposta ora apre
con `Categoria: SE1520`, cioe la classe della **nuova** tassonomia derivata dal codice materiale,
non piu una legacy e non piu assente. Gli altri tre punti restano invariati, tempo 41,7s.

**Difetto lingua (test di Andrea Maggiora del 29/09):** alla domanda "Su che conto viene
contabilizzato questo materiale?" l'agente premetteva *What is your current language preference?*.
Corretto con una regola in testa al blocco LANGUAGE OVERRIDE che vieta di chiedere la lingua e
impone di dedurla dall'ultimo messaggio dell'utente. Verificato: risponde direttamente in italiano.
Da rieseguire come secondo caso di regressione, in sequenza di due domande nella stessa sessione.

**Difetto aperto trovato il 29/09 alle 13:46:** alla richiesta costituita da una sola parola
italiana ambigua ("laboratorio") la risposta arriva **in inglese e senza etichetta MMD |**:
"Could you provide a bit more context about laboratorio?". La richiesta di chiarimento non passa
dal child: la produce l'orchestratore, che in quel percorso ignora sia la regola della lingua sia
quella dell'etichetta. Le regole nel child non possono risolverlo. Da correggere nelle istruzioni
del **parent**, dove restano circa 68 caratteri liberi su 8000.

**09/10/2026 - il caso canonico perde la seconda sezione, e NON e' intermittenza.** Rilanciato
**quattro volte** in F&O su RSSE, sessione nuova ogni volta (ricaricando F&O, non riusando la
conversazione):

| | SE152002.0001 | SE152001.0001 | SE105001.0001 | due sezioni |
|---|---|---|---|---|
| 4 run su 4 | sempre presente | **mai** | 1 volta su 4 | **mai** |

Avevo detto a Raffaele che era intermittenza dopo **una** osservazione: sbagliato. Una
osservazione non distingue "capita a volte" da "capita sempre"; servono le ripetizioni, e qui
bastavano tre.

**La prova decisiva e' la run history, non la risposta.** Run del 09/10 alle 10:04:35, Succeeded
in 22s: il flusso `RINA - Search Items` restituisce **tutto**, e in `Rispondi all agente` si legge

- `otherlegalentities`: `SE152001.0001 | Manutenzione attrezzature di produzione | Service |
  category Mainten | NOT released in the user company - released in PATR`
- `company`: `RSSE`
- `items`: **35 articoli**
- `categories`: solo descrizioni, zero codici in vecchia nomenclatura (conferma del fix al flusso)

L'agente ne mostra 3-6 e **omette del tutto il blocco dei richiedibili**. Il dato c'e', il modello
lo butta.

**Ipotesi con un appoggio nei dati:** la perdita e' legata alla **dimensione** del payload. Le
ricerche con pochi risultati (ISO 9001, reagenti chimici) conservano le due sezioni; quelle con
molti (manutenzione attrezzature con 35 articoli, consulenza informatica) le perdono. Se si
conferma, la leva e' ridurre a monte gli articoli restituiti per pertinenza — che pero' contrasta
con la riga 3 del registro UAT, dove i tester chiedono di non applicare soglie arbitrarie. E' una
scelta da portare a RINA, non da decidere nel flusso.

**How to apply:** quando una risposta omette qualcosa, aprire la run del flusso e leggere
`Rispondi all agente` **prima** di toccare istruzioni o flusso. Distingue in un minuto "il dato non
c'e'" da "il modello lo scarta", e sono due problemi con due soluzioni diverse.
