---
name: copilot-studio-browser-quirks
description: "Practical quirks when driving Copilot Studio / agent flow designer via the user's Chrome (hidden tab, optional inputs, expression editor)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-09-18T15:26:27.481Z
---

Copilot Studio (env rina-uat-clm) automation via Chrome, learned 2026-09-09:
- Copilot Studio pages render blank/stuck while the tab is hidden: ask the user to keep the tab in front (or open a new tab, which becomes active). Tab groups get lost when the user closes tabs; recreate with tabs_context createIfEmpty.
- Tool inputs "Dynamically fill with AI" cannot be left empty by the orchestrator ("Could not assign input X"): do not expose optional flow inputs as tool inputs; let the flow default them.
- Questions generated for tool inputs come from the input descriptions (orchestrator), not from the child agent instructions: put the language rule in each description.
- Agent flow designer: action Code view is read-only; expression editor rejects paths not in the action schema (use string(outputs(x)?['body'])) and pasted text with trailing newline; first "Add" sometimes does not apply, verify the field.
- Fin & Ops connector Delete record on PurchaseRequisitionHeaders: Object id = RequisitionNumber only (no dataAreaId).
- Expression editor: first "Add" often shows "The expression is invalid" for a valid expression; press Add again. Always verify advanced params (Filter Query) persisted via Code view or the run inputs: a missing filter on Workers scans the whole table and times out the flow (agent limit 100 s, Express 120 s).
- Fin & Ops connector latency on rina-uat-clm: first call ~20 s warm-up, header POST 13-20 s, line POST 10-52 s; keep the flow minimal.

- Instructions editor: ctrl+End does NOT move the caret to the end; typed text can land mid-word. Place the caret with the Selection API on the editor's last element, type one test character and verify its index before typing the real block. If text lands wrong, use Cancel then Discard.
- Duplicate final message after a child agent: "Don't respond" completion does not stop it. Fix (Microsoft guidance): child returns `answered` + `interactionSummary` with parent-facing descriptions, and the parent has an instruction that reads `answered` before replying. Scope that instruction to the agents that talk to the user: a silent child that uses `answered` to mean "resolved" would stop getting its answers delivered.
- Flow used as agent tool with per-user identity: set the flow's connection reference (flow Overview > Connection references > Manage) to "Provided by run-only user" and republish both flow and agent; each user then gets "Connect to continue". A specific connection there makes every write run as that account. `InvokerConnectionOverrideFailed ... X-MS-APIM-Tokens` means the invoker token did not reach the flow.

- Fin & Ops connector "Lists items" $filter: a "&" inside a string literal (e.g. a description "Finance & Operations") truncates the filter ("unterminated string literal"). Never filter on free text: filter on codes/dates and compare text in a Query (Filter array) action.
- Editing agent flows reliably: "Copy entire action" on a top-level Condition/Scope puts JSON {nodeId, serializedValue, allConnectionData,...} in the clipboard; modify it in JS, hook navigator.clipboard.readText, delete the node, "Paste an action", Save draft immediately. In a Query "where", literal strings lose their quotes: use concat('Fail','ed') style.

- 2026-09-18: Copilot Studio defaults to a "new experience" where flows and child agents do not open. Load a flow at /environments/<env>/agent-flows/<flowId> (it opens classic), then history.pushState + popstate to /bots/<botId>/agents/<childId>/details or /bots/<botId>/overview to stay in classic.
- Backup without downloads: a real "Copy entire action"/"Copy trigger" click (or navigator.clipboard.writeText right after a click) fills the Windows clipboard only when the tab is visible; then PowerShell Get-Clipboard -Raw saves it. Dataverse Web API returns "user is not a member of the organization" for this account. Backups are in Documents\RINA_backup_2026-09-18.

- 2026-10-05: un run di **Evaluation** avanza solo mentre la scheda e' visibile: se la finestra passa dietro il run si congela a meta' (fermato a 20/24, 9/24, 8/24) e riparte da solo quando torna in primo piano. 24 casi richiedono ~17 minuti di finestra davanti, senza interruzioni.

**Why:** these cost hours in the RINA RDA flow work. **How to apply:** check these before automating Copilot Studio again. Related: [[copilot-studio-language-rule]].

## 19/10/2026 - input riempiti dall'AI: la causa vera e' la lista chiusa

L'agente ogni tanto chiede "In quale lingua stai scrivendo il messaggio?" invece di rispondere.
L'input `UserLanguage` dello strumento Search Items e' su **Dynamically fill with AI**.

Due cose accertate guardando la configurazione:
- `Advanced > Should prompt user` era **gia' deselezionato** e non impedisce la domanda: la
  domanda nasce dall'orchestratore, non dal prompt dell'input.
- `Fill using = Custom value` non e' una via d'uscita: l'unica variabile di lingua disponibile e'
  `System.User.Language`, che e' il **locale dell'utenza**, non la lingua del messaggio. Non
  esiste nessuna `Activity.Locale` nel selettore.

**Quello che ha funzionato**: la descrizione dell'input imponeva una **lista chiusa** di lingue
("it, fr, pt, es, zh, en"). Fuori da quell'insieme, o in caso di dubbio, il modello esita e
chiede. Riscritta in forma generica ("ISO 639-1 code ... whatever that language is ... always
provide a value and never ask the user; if the message is too short, repeat the language of the
previous user message, and if there is none, use it"). Due prove su due in italiano senza
domanda. Rollback in `ROLLBACK_userlanguage_descrizione.md`.

**Why**: vale la regola generale sulla lingua di [[copilot-studio-language-rule]] - le liste
chiuse non sono solo poco eleganti, fanno esitare il modello e generano domande all'utente.
**How to apply**: ogni input AI-filled va descritto in modo che una risposta sia *sempre*
determinabile, con un ripiego esplicito. Mai elencare i valori ammessi quando l'insieme reale e'
aperto.
