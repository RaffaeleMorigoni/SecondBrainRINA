---
name: rina-upn-non-email
description: Gli strumenti dell'agente devono passare User.PrincipalName, non User.Email - sono due attributi diversi dello stesso account
metadata:
  type: reference
---

Nei tool di Copilot Studio che chiamano i flussi RINA, l'input `UserEmail` va valorizzato con
**`System.User.PrincipalName`**, mai con `System.User.Email`.

Sono due attributi diversi dello stesso account Entra:
- `User.PrincipalName` -> **`C_110912@rina.org`**, l'UPN, che e cio che F&O conosce come
  `PersonUsers.UserEmail`
- `User.Email` -> `raffaele.morigoni@ext.rina.org`, l'attributo *mail* del profilo, che **in F&O
  non esiste**

Con `User.Email` tutta la catena si rompe al primo passo: nessun PersonUser -> nessuna societa ->
filtri con `dataAreaId eq ''` -> zero risultati. L'agente non va in errore: ripiega sulle
categorie merceologiche dell'MCP e sulla knowledge, e **sembra funzionare** (Answer quality 83%)
mentre il Meaning match crolla (21%), perche le risposte sono plausibili ma non contengono i
codici attesi.

**Errore commesso il 07/10/2026.** Ricollegando i quattro flussi all'agente ho scelto `User.Email`
perche era la prima variabile plausibile dell'elenco, senza controllare come erano configurati
prima. La risposta era gia scritta in [[rina-fo-user-worker-prerequisite]]: *"L'agente RDA ricava
il preparatore cosi: System.User.PrincipalName -> PersonUsers.UserEmail -> PartyNumber -> ..."*.
Ne sono seguite due ore di diagnosi sbagliata, fino a ipotizzare che l'utenza non fosse censita
(lo era, usata per mille test) e a proporre un'utenza fissa, bloccata due volte dal classificatore
di sicurezza - giustamente, perche equivaleva a far interrogare F&O a tutti con un'identita altrui.

`User.PrincipalName` **non compare** cercando "User." nel selettore delle variabili di sistema:
va cercato per "Principal".

**Why:** l'errore non da errore, da risposte verosimili e sbagliate - il tipo piu costoso da
diagnosticare.
**How to apply:** ogni volta che si configura o si ricollega un tool con l'input `UserEmail`,
impostare `User.PrincipalName`. E prima di ricreare una configurazione **leggere i propri appunti
e la wiki del progetto**, invece di dedurre dal contesto.
