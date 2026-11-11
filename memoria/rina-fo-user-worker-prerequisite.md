---
name: rina-fo-user-worker-prerequisite
description: "RINA F&O (rina-uat-clm): ogni utente dell'agente RDA deve avere un lavoratore collegato all'utenza, altrimenti la RDA non parte"
metadata:
  type: project
  originSessionId: 202606c8-1844-495c-8cc0-e9936da067de
  modified: 2026-09-10T09:54:18.638Z
---

In rina-uat-clm l'identità delle utenze F&O è `C_<matricola>@rina.org` (es. C_110912 = Raffaele Morigoni, C_110910 = Chiara D'Ambrosio). L'agente RDA ricava il preparatore così: System.User.PrincipalName → PersonUsers.UserEmail → PartyNumber → Workers.PersonnelNumber → HcmWorkerBiEntities.SourceKey (RecId). Se l'utenza non ha una persona collegata (PersonUsers e AVADirPersonUsers vuoti) o non esiste il lavoratore con quella matricola, Originator resta vuoto e F&O rifiuta la testata con "Field 'Preparer' must be filled in".

**Why:** il test del 10/09/2026 con Chiara D'Ambrosio è fallito per questo, non per un bug del flusso; è lo stesso prerequisito che il 09/09/2026 richiese di creare il lavoratore 110912 per la mia utenza.

**How to apply:** prima di far provare l'agente a un nuovo utente RINA, verifica via OData che `PersonUsers?$filter=UserEmail eq 'C_<matricola>@rina.org'` restituisca un PartyNumber e che esista il worker con quella matricola. Se manca: crea il lavoratore da Human resources > Workers > New (matricola = numero dell'utenza, legal entity RSPA) e crea il legame con POST su `PersonUsers` (UserId, PartyNumber, ValidFrom, ValidTo 2154-12-31T23:59:59Z); UserEmail e PersonName sono derivati e si popolano in lettura. Fatto per Chiara il 10/09/2026 (matricola 110910, party 000901821). Nota: `contains()` e `startswith()` non sono supportati su queste entità, filtra con `eq` o scansiona (Workers ha 27k righe, ~30 s per pagina). Vedi [[copilot-studio-browser-quirks]].
