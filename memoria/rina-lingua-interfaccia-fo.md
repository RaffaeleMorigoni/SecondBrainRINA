---
name: rina-lingua-interfaccia-fo
description: "L'italiano ha la precedenza: basta un segnale italiano (locale F&O o domanda) perche' la risposta esca in italiano - l'unico caso rotto e' locale it + domanda non italiana"
metadata:
  type: project
---

**Matrice provata il 30/10/2026**, stesso giorno, stesso codice `SE108002.0001`, sessione nuova
ogni volta, RSSE, `C_110912@rina.org`, stesse istruzioni (padre 7.859, figlio 29.447).
Unica variabile: Opzioni utente > Preferenze > Lingua.

| | domanda IT | domanda EN |
|---|---|---|
| **UI `it`** | (coerente) | **italiano** ❌ |
| **UI `en-US`** | **italiano** ✅ | **inglese** ✅ |

**La regola e': l'italiano ha la precedenza.** Basta **un solo** segnale italiano — il locale
*oppure* la domanda — perche' la risposta esca in italiano. Per avere inglese devono essere
inglesi **tutti e due**.

**Non dire "comanda il locale"**: l'ho scritto stamattina ed era troppo forte. Con UI inglese e
domanda italiana la risposta e' italiana, cioe' li' la domanda vince. Due celle su tre sono
corrette.

**Il dato e' sempre giusto.** La descrizione segue sempre la lingua della domanda
(`High-altitude mechanical systems maintenance` con domanda EN, `Manutenzioni impianti meccanici
in quota` con domanda IT), in entrambi i locali: flusso e `ProductTranslations`
([[rina-traduzioni-prodotto-esistono]]) funzionano. A lasciarsi trascinare e' solo la **prosa di
contorno**. Lo confermano i residui misti nella stessa risposta: `Tipo: service`,
`Legal entity: RSSE` dentro un testo italiano — non c'e' template, le etichette sono improvvisate
ogni volta.

**Portata reale del difetto**: colpisce **solo i tester italiani che provano in inglese** — cioe'
la segnalazione FOFI. Un utente francese o portoghese che scrive nella sua lingua non e' toccato.
Non descriverlo come difetto generale sulle lingue.

**Perche' gli otto tentativi dal prompt non sono serviti**: [[rina-padre-copia-non-traduce]],
[[rina-formato-fo-non-governabile]]. Nel pannello di test il locale non esiste, ed e' per questo
che li' l'inglese usciva sempre giusto.

**Regola per le segnalazioni**: dichiarare sempre la lingua di interfaccia di chi prova,
altrimenti i casi non sono confrontabili ([[rina-suite-regressione]]).

Prove in `plans/prove_uat/20261030_interfaccia_*`, dettaglio in
`plans/20261030_test_lingua_interfaccia.md`.
