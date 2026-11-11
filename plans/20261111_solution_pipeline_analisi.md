# Solution + pipeline: cosa si può pacchettizzare e cosa no

**Data:** 11/11/2026
**Domanda:** nella solution c'è un agente managed (l'orchestratore di F&O). Problemi?
**Risposta breve:** sì, tre. Il primo impone un cambio di disegno.

Riferimenti verificati su Microsoft Learn (11/11/2026), citati in fondo.

---

## Problema 1 — L'orchestratore padre non è pacchettizzabile. Per progetto.

### I fatti

Microsoft distribuisce gli agenti Dynamics 365 **come managed solution**
(*"Microsoft ships Dynamics 365 agents as managed solutions"*). L'orchestratore
`Copilot for finance and operations apps` è quindi un componente **managed di Microsoft**,
già installato in ogni ambiente.

Noi le sue istruzioni le abbiamo modificate. Quella modifica, per come funziona Dataverse,
**non sta in nessuna solution nostra**: vive in un **unmanaged layer** sopra il managed di
Microsoft. La documentazione di Copilot Studio ha una sezione apposita che si chiama
*"Remove an unmanaged layer from a managed agent"* — cioè: il caso è previsto, e l'unica
operazione offerta è **rimuoverlo**, non esportarlo.

### Le tre conseguenze

**a) La pipeline non lo trasporta.** Un unmanaged layer non è un artefatto deployabile.
Non c'è modo di far viaggiare le istruzioni del padre dentro una solution.

**b) In produzione un unmanaged layer batte qualunque managed tu importi.** Dataverse
applica *"top wins"*: il layer unmanaged (*Active*) sta sopra tutti i managed. È il sintomo
documentato come *"Changes aren't effective after solution import"*: se in prod qualcuno ha
già toccato a mano una cosa, il tuo import si installa senza errori e **non ha effetto**.
Questa è la trappola peggiore, perché la pipeline dice verde.

**c) Blocca gli aggiornamenti di Microsoft.** Learn è esplicito: un unmanaged layer su un
componente managed *"override updates from the source managed solution"* e
*"prevent changes from being reflected correctly after deployment"*. L'app Copilot for F&O
viene aggiornata da Microsoft con cadenza propria: sui campi che abbiamo toccato i loro
update non arriveranno, o arriveranno in conflitto. In più l'unmanaged layer può
**impedire la disinstallazione** della managed solution sottostante.

### La conseguenza sul disegno

**Non mettere l'agente padre nella solution. Non provarci.** La frontiera corretta è:

| nella solution (pipeline) | configurazione d'ambiente (manuale) |
|---|---|
| agente figlio | istruzioni dell'orchestratore padre |
| flussi Power Automate | collegamento padre → figlio (connected agent) |
| connection reference | descrizione del connected agent |
| environment variable | knowledge |
| eventuali connettori custom | modello selezionato, Express mode |
| | condivisione agli utenti |

Non è un compromesso al ribasso: è la separazione che il prodotto impone. Quella
configurazione **appartiene all'ambiente**, non all'artefatto.

**Quindi:** le istruzioni del padre vanno versionate **come file di testo in questo repo**,
con una checklist di applicazione post-deploy. È l'unica forma di "deploy" e l'unica forma
di rollback che quella parte avrà mai.

---

## Problema 2 — Il collegamento padre→figlio resta manuale, e con esso il routing

Il figlio arriva in prod via pipeline, e gli ID si preservano. Ma **"Connected agents" è
configurazione sul padre**, cioè sul componente managed: va rifatta a mano in produzione.

Il punto non banale: la documentazione dice che l'orchestratore decide se delegare in base a
**nome e descrizione del connected agent** più le istruzioni del primario. Quindi quella
descrizione non è etichetta, è **logica di routing** — e viaggia a mano, con il rischio di
arrivare in prod diversa da UAT. Va nella baseline testuale insieme alle istruzioni del padre.

Da qui segue anche il test obbligatorio in prod: **provare come utente finale, non come
maker.** Learn lo raccomanda esplicitamente — condividere il padre con un utente *senza*
condividere il figlio, e verificare che il padre riesca comunque a invocarlo.

---

## Problema 3 — Anche per il figlio, parte della configurazione non viaggia

Confermato su Learn, lato export/import agenti:

- Le **istruzioni dell'agente (main prompt) sì**, viaggiano. Buona notizia per il figlio.
- **Non** viaggiano: Conversation ID, CDS Bot ID, Environment ID, icona, dettagli dei canali.
- **L'autenticazione utente va riconfigurata** dopo l'import. Su questo agente è critico:
  tutta la catena verso F&O dipende dall'identità dell'utente (UPN).
- **L'agente importato non è pubblicato**: *"Imported agents aren't published automatically"*,
  e va pubblicato prima di poter essere condiviso.
- Le **connection reference** vanno ripuntate su connessioni valide dell'ambiente.
- I **flussi vanno verificati accesi**: l'import non li attiva necessariamente.
- Le **knowledge** possono non essere incluse (*"might not be included if they're stored as
  separate resources or Dataverse tables"*). Da ricaricare e verificare che diventino *Ready*:
  caricate non basta, devono completare l'indicizzazione.

Tre vincoli operativi da sapere prima:

- **Non si esporta una solution managed.** L'ambiente sorgente deve restare unmanaged.
- **Niente punti (`.`) nei nomi dei topic**, o l'export fallisce.
- Non rimuovere componenti del figlio (topic, ecc.) direttamente dalla solution: fa fallire
  export e import.

---

## La questione che va decisa prima di configurare la pipeline

Oggi stiamo costruendo **direttamente in UAT**. Se UAT diventa l'ambiente sorgente della
pipeline, allora:

- in UAT la solution resta **unmanaged per sempre** (non si esporta una managed);
- di conseguenza **UAT non riceverà mai la managed**, e non è più un collaudo fedele a prod:
  è il dev.

Due strade, da scegliere con RINA:

1. **UAT = dev/source, PROD = target.** Semplice, due ambienti, zero costi aggiuntivi.
   Si accetta che il collaudo avvenga su un ambiente unmanaged, diverso per natura da prod.
2. **DEV → UAT → PROD.** UAT diventa un vero ambiente di collaudo su managed, e i difetti da
   layering si scoprono lì invece che in produzione. Costa un ambiente e un passaggio in più.

Se il rilascio in produzione è vero, la 2 è la scelta tecnicamente corretta. La 1 è
accettabile solo se si mette in conto che il primo import managed in prod è anche il primo
test di quell'import.

C'è anche un prerequisito di piattaforma da verificare: la solution attuale.
Gli agenti Copilot Studio **nascono nella solution Default**, che non serve per l'ALM. Se
figlio e flussi sono ancora lì — probabile — vanno spostati in una solution custom con il
nostro publisher prefix, scelto adesso e mai più cambiato.

---

## Cosa verificherei in ambiente, nell'ordine

1. **Il padre ha già un unmanaged layer?** Copilot Studio → agente → ⋮ → Advanced →
   *See solution layers*. Ci aspettiamo di trovarlo, ma va visto: dice anche cosa esattamente
   abbiamo stratificato sopra Microsoft.
2. **Dove stanno figlio e flussi oggi** — Default o solution custom.
3. **I flussi usano connection reference o connessioni dirette?** Le connessioni dirette non
   si deployano: è il classico flusso che in prod punta a nulla.
4. **Lo stesso layer esiste già in produzione?** Se qualcuno ha toccato il padre in prod,
   va saputo prima, non durante l'import.
5. **Publisher prefix** da concordare.

---

## Fonti

- [Missing dependencies error during solution import](https://learn.microsoft.com/troubleshoot/power-platform/dataverse/working-with-solutions/missing-dependency-on-solution-import) — *Avoid modifying components in managed solutions*
- [Solution layers](https://learn.microsoft.com/power-platform/alm/solution-layers-alm) — unmanaged layer, "top wins"
- [Changes aren't effective after solution import](https://learn.microsoft.com/troubleshoot/power-platform/dataverse/working-with-solutions/changes-not-effective-solution-import)
- [Create and manage solutions in Copilot Studio](https://learn.microsoft.com/microsoft-copilot-studio/authoring-solutions-overview) — *Remove an unmanaged layer from a managed agent*
- [Export and import agents using solutions](https://learn.microsoft.com/microsoft-copilot-studio/authoring-solutions-import-export) — cosa viaggia e cosa no
- [Deploy Dynamics 365 agents](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/copilot/agent-deployment) — *Microsoft ships Dynamics 365 agents as managed solutions*
- [Add a Copilot Studio agent as a connected agent](https://learn.microsoft.com/microsoft-copilot-studio/agents-experience/add-agent-connected) — routing da nome e descrizione
