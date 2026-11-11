# 2418 — il fix sulla mancata invocazione

Pubblicato il **28/10/2026 alle 17:45** sull'orchestratore
(*Copilot for finance and operations apps*). Istruzioni da **7.775 a 7.859 / 8.000**.

---

## L'ipotesi, che e' nuova

Le tre regole provate il 21/10 **aggiungevano divieti**: non nominare codici a memoria, i file in
Knowledge servono al mapping, rispondi nella lingua dell'utente. Nessuna ha retto a freddo.

Oggi lo screenshot di RSSE su `EX102002.0001` mostra una cosa che prima non avevo messo a fuoco:
la risposta **non e' inventata**, e' **letta dalla pagina aperta**. L'utente aveva davanti la griglia
*Released product details*, e l'agente ha restituito il `Product name` della riga selezionata,
in italiano, dentro una risposta in inglese.

E le istruzioni **gli davano il permesso di farlo**. Il blocco SCOPE diceva:

> For any other request (general Dynamics 365 how-to, **questions about the app data or pages**,
> troubleshooting, greetings or any other topic) behave freely… answer normally with your standard
> capabilities, product knowledge and **the app data you can access**.

Una domanda come *"item number ex102002.001"*, posta mentre quella riga e' a video, e' classificabile
come **domanda sui dati della pagina**. A quel punto l'agente non sta violando nessuna regola:
sta prendendo il ramo libero, che e' li' apposta.

**Per questo i divieti non servivano a niente: il problema non era un divieto mancante, era un
permesso di troppo.**

---

## Cosa ho cambiato

Due sostituzioni, nessuna aggiunta altrove.

**1. Chiuso il ramo libero** (la radice del problema):

| | |
|---|---|
| prima | `questions about the app data or pages,` |
| dopo | `questions about the app data or pages other than item, product or MMD codes,` |

**2. Unificata la regola sulle fonti**, che prima parlava solo della Knowledge:

| | |
|---|---|
| prima | `The material code files in Knowledge are for legacy-to-new mapping only, never a list of usable codes: run a search before naming any code, including one just read there.` |
| dopo | `Never answer about an item, product or MMD code from what is already in view or in Knowledge: those are page context and legacy-to-new mapping only, never a source of codes or descriptions. Always run a search first.` |

La seconda tiene il meccanismo che il 21/10 aveva funzionato — **dire a cosa serve una fonte**,
non vietarla in astratto — e lo estende dalla Knowledge ai dati a video, che erano la fonte
davvero usata.

Copia di servizio del testo precedente:
`backup/20261028_istruzioni_padre_PRIMA_fix_invocazione.txt`

---

## Come si verifica, e perche' non l'ho verificato io

**Il test va fatto in F&O, non nel pannello di test**, e con **una riga prodotto a video**: e' la
condizione che innescava il difetto. Nel pannello di test il contesto pagina non esiste, quindi
non proverebbe niente.

Caso minimo:

1. F&O, societa' **RSSE**, pagina *Released product details* (`EcoResProductDetailsExtendedGrid`);
2. selezionare la riga `EX102002.0001` — *Deposito cauzionale*;
3. chiedere al Copilot, **in una lingua diversa dall'italiano**: *"item number EX102002.0001"*;
4. atteso: parte una run del flusso `RINA - Search Items` e la descrizione esce **nella lingua della
   domanda**, non in italiano.

**Il controllo che conta e' la run del flusso**, non la bellezza della risposta: se non c'e' run,
il fix non ha preso, qualunque cosa abbia scritto l'agente.

**E va ripetuto a freddo il giorno dopo.** Il 21/10 una prova riuscita subito dopo la pubblicazione
si e' sciolta nella notte. Una prova fatta adesso non chiude il 2418.

---

## Cosa resta se non basta

In ordine, come il 22/10:

1. **la descrizione dello strumento** `RINA - Search Items` sul padre: e' il testo che l'orchestratore
   legge per decidere se invocarlo;
2. **il timeout** (2494): finche' la ricerca non si completa, l'agente ha comunque un motivo per
   ripiegare su altro. I due difetti si alimentano a vicenda.

Da tenere separati: **una modifica alla volta**, altrimenti non si sa quale ha funzionato.
