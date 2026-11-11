---
name: copilot-studio-istruzioni-limite-8000
description: Il campo Instructions di Copilot Studio ha un limite di 8000 e tronca in silenzio; si misura come caratteri + numero di newline
metadata:
  node_type: memory
  type: reference
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-07T13:20:24.349Z
---

Il campo **Instructions** dell'agente Copilot Studio ha un limite di **8.000**, e il conteggio e' `lunghezza del testo + numero di newline` (le righe contano come CRLF). Verificato esattamente il 07/10/2026: un testo di 7.842 caratteri con 158 newline faceva somma 8.000 tondi, ed era il punto preciso in cui il salvataggio aveva tagliato.

**Tronca in silenzio.** Nessun errore, nessun avviso, nessun attributo `maxlength` sul campo. Il testo in eccesso sparisce a meta' frase e la sezione successiva si fonde con quella tagliata. E' cosi' che si era corrotto il blocco `ANSWER FORMAT` (un blocco precedente finiva con "cannot be" e ci si attaccava sopra), e quella misura della suite girava su istruzioni mutilate senza che si vedesse nulla.

**Come scrivere nel campo senza rompere tutto:**
- Calcolare sempre `testo.length + newline` prima di salvare e stare sotto 8.000 con margine.
- Non usare `document.execCommand('insertText', ...)` con blocchi lunghi: distrugge il contenuto dell'editor.
- Usare la clipboard: prima il **click reale** nel campo (serve il focus del documento, altrimenti `navigator.clipboard.writeText` fallisce con "Document is not focused" e Ctrl+V incolla quello che c'era prima nella clipboard dell'utente), poi `writeText`, poi Ctrl+A e Ctrl+V.
- L'editor e' virtualizzato: `innerText` e il TreeWalker possono restituire meno di quello che c'e'. Verificare sempre dopo il salvataggio, scrollando fino in fondo alla card Instructions, che la coda finale sia ancora li'.

Verificare la coda e' il controllo piu' economico: se l'ultima riga del FINAL CHECK non c'e' piu', il salvataggio ha tagliato. Vedi [[rina-suite-regressione]] e [[copilot-studio-browser-quirks]].
