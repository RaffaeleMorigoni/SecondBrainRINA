---
name: rina-condivisione-agente-omonimie
description: "Aggiungere utenti alla condivisione dell'agente: il picker cerca solo per nome proprio, e nella directory RINA ci sono molti omonimi"
metadata:
  node_type: memory
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-10-28T16:23:07.738Z
---

**Dove:** agente padre in Copilot Studio → menu `…` → **Share** → campo *Add a name, group, or email*.
Si assegna *End user access* (compare come **Viewer**), poi **Share**: il banner dice
*"Successfully changed the sharing settings for your agent"* e il contatore *Existing users* sale.

**Il picker non cerca per cognome.** Digitare `bordallo` non restituisce nulla; la persona si trova
solo con il nome proprio (`joao vitor`). Vale per i cognomi multipli brasiliani e spagnoli, che in
RINA sono tanti.

**Omonimie da non sbagliare** (verificate il 28/10/2026):
- tre *Joao Vitor*: DOS SANTOS BORDALLO DUTRA (`joaovitor.dossantos@rina.org`, quello giusto),
  DE OLIVEIRA GOMES, FERNANDES DE BRITO;
- due *Ana(maria)*: MANAILA (`Anamaria.Manaila@rina.org`, quella giusta) e GASTELUT VAILLO.

Controllare sempre l'indirizzo nel suggerimento prima di selezionare.

**Per verificare chi c'e' gia' in lista** conviene leggere il testo del pannello invece di scorrerlo
a occhio: con 50 voci la lista e' lunga e l'ordine alfabetico sui nomi stranieri inganna.

Il 28/10 la lista e' passata da 50 a 52 aggiungendo i due tester che erano bloccati
(vedi [[rina-accesso-agente-vs-ambiente]]).
