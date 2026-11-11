# Il secondo turno regge. A non funzionare e' la ricerca in francese

Prova in F&O, 30/10/2026 ore 15:20 circa. RSSE, sessione nuova, interfaccia italiana,
`C_110912@rina.org`, modello GPT Chat Latest appena pubblicato.

## Cosa ho fatto

Una sola sessione, due turni, **la stessa identica domanda** nelle due lingue.

**Turno 1, francese:**
> Je dois acheter un service de maintenance d'equipements. Quel code MMD dois-je utiliser ?

> MMD | Aucun code MMD correspondant precisement a un service de maintenance d'equipements
> n'a ete identifie parmi les articles disponibles pour RSSE.

**Turno 2, italiano, stessa sessione:**
> Devo acquistare un servizio di manutenzione attrezzature. Quale codice MMD devo usare?

> MMD | SE152002.0001 Manutenzione attrezzature tecnologiche

## Le due conclusioni

**1. Il secondo turno funziona.** Era il punto aperto che la suite non puo' misurare, perche'
e' a risposta singola. Qui il secondo turno non solo ha risposto: e' **l'unico dei due che ha
trovato il codice**. Il sospetto di ieri — Chat Latest che dice *"aucun MMD trouve"* al
follow-up — non si e' riprodotto in questa forma.

**2. Il fallimento francese non e' del secondo turno ne' del modello: e' della ricerca.**
Al turno francese la risposta e' arrivata **senza che partisse** `Operazione in corso...`;
al turno italiano il flusso e' girato per circa un minuto e ha trovato. Stessa domanda,
stessa sessione, stesso modello: cambia solo la lingua delle parole chiave.

E' esattamente [[rina-ricerca-parole-italiano-iniziaper]]: la ricerca interroga
`ProductSearchName`, che e' **in italiano**. `maintenance` non incontra `Manutenzione`.
Le traduzioni esistono in `ProductTranslations` ([[rina-traduzioni-prodotto-esistono]]) ma
**la ricerca non le usa**: servono a scrivere la risposta, non a trovarla.

## Perche' conta

La forma francese e' **perfetta**: prefisso `MMD | `, frase in francese corretto, nessuna
prosa italiana. Il lavoro sulla lingua ha funzionato. Ma un utente francese che cerca per
parole **non trova niente**, e riceve un "nessun codice" ben scritto al posto del codice che
esiste.

Un tester francofono leggerebbe questo come *"l'agente non conosce i codici"*. E' invece un
difetto di **copertura linguistica della ricerca**, e va segnalato come tale.

## Cosa resta da capire

Non ho verificato in Power Automate se al turno francese il flusso sia partito a vuoto o non
sia partito affatto: ho osservato solo l'assenza dell'indicatore a video e la rapidita' della
risposta. Va chiuso guardando la run — sono due difetti diversi e si correggono in punti
diversi.

Prove: `prove_uat/20261030_FR_turno1_nessun_codice.png`,
`prove_uat/20261030_IT_turno2_stessa_domanda_trova_il_codice.png`.

---

# VERIFICA IN POWER AUTOMATE — il flusso parte, ed e' la prova definitiva

Flusso `RINA - Search Items`, `28-day run history`. Due run, non una.

| | **FR — 15:40:02** | **IT — 15:41** |
|---|---|---|
| esito | `Succeeded` | `Succeeded` |
| durata totale | **00:00:07** | **00:00:54** |
| `Filtra parole` (input = output) | `maintenance`, `equipements`, `service` | `manutenzione`, `attrezzature`, `servizio` |
| `Lista prodotti 1-4` | **1s, 0.6s, 1s, 1s** | **48s, 48s, 47s, 47s** |
| data limit (Express mode) | meno dell'1% | 2% |

## Cosa dimostra, esattamente

**Il flusso e' partito anche in francese**, ed e' arrivato in fondo senza errori. La mia lettura
di prima — *"non e' partito"* — era sbagliata: avevo dedotto dall'assenza dell'indicatore a
video e dalla rapidita'. Era proprio la rapidita' il segnale, ma di un'altra cosa.

**L'orchestratore estrae le parole chiave nella lingua dell'utente e le passa cosi' come sono.**
`Filtra parole` riceve `maintenance / equipements / service` e restituisce le stesse tre parole:
nessuna traduzione, in nessun punto della catena.

**Le quattro query girano a vuoto in un secondo.** 1s contro 48s non e' una differenza di
prestazioni: e' la differenza fra una query che non trova niente da scandire e una che lavora.
Il campo interrogato e' `ProductSearchName`, che e' in italiano
([[rina-ricerca-parole-italiano-iniziaper]]).

## Dove va corretto

**Non nel prompt.** Il padre sta facendo il suo mestiere: estrae le parole dalla domanda.
Il punto di rottura e' fra l'estrazione e la query.

Due strade, da valutare separatamente:
1. **tradurre le parole chiave in italiano prima della query** — nel flusso, fra `Filtra parole`
   e `Lista prodotti`, o chiedendo al padre di passarle gia' in italiano;
2. **cercare anche su `ProductTranslations`** — le traduzioni esistono in 7 lingue
   ([[rina-traduzioni-prodotto-esistono]]) e oggi servono solo a scrivere la risposta.

La seconda e' piu' pulita ma tocca le query, che sono gia' il collo di bottiglia
([[rina-search-items-query-sequenziali]], e il 2494). La prima e' piu' economica: tre parole da
tradurre contro quattro query da allargare.

Prove: `prove_uat/20261030_PA_filtra_parole_FR_maintenance.png`,
`prove_uat/20261030_PA_filtra_parole_IT_manutenzione.png`.
