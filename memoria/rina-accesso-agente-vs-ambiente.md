---
name: rina-accesso-agente-vs-ambiente
description: "Accesso a F&O e condivisione dell'agente sono due autorizzazioni distinte: chi ha solo la prima riceve 'You don't have access to talk to this bot'"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-28T16:02:37.939Z
---

**Sono due cose diverse, e averne una non implica l'altra:**

1. l'accesso all'**ambiente F&O** (lo da' chi gestisce Dynamics);
2. la **condivisione dell'agente** in Copilot Studio — menu `…` → **Share**, dove c'e' la lista
   degli utenti con ruolo *Viewer*.

Chi ha solo la prima riceve, aprendo Copilot in F&O:
**`You don't have access to talk to this bot, contact the owner.`**

**Caso reale (28/10/2026).** João Bordallo scrive *"Although access has already been granted…
I get a connection error"*: gli era stato dato l'ambiente, **non** l'agente. Nella condivisione
(50 utenti) c'erano Nistor, Lena Lensha, Maria Poulaki, Mantese, Emanuela, Enrico — **mancavano
João e Anamaria Manaila**, quest'ultima richiesta da Enrico due giorni prima.

**Come si verifica:** aprire Share e confrontare la lista con i nomi delle mail. Si trova in un
minuto e spiega sessioni che altrimenti sembrano difetti dell'agente.

**Attenzione a non confondere i due sintomi:**
- `You don't have access to talk to this bot` → **manca la condivisione** (questo file);
- `Open connection manager to verify your credentials` → **e' il server MCP**, che non eredita le
  credenziali dell'autore (vedi [[rina-mcp-dynamics-solo-nel-test]]).

Si vedono solo in `Analytics → Sessions`
([[rina-monitoring-sessioni-dove-guardare]]): non lasciano traccia nelle run di Power Automate,
perche' il flusso non parte nemmeno.
