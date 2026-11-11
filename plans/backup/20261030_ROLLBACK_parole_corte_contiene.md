# ROLLBACK — ramo "parole corte" delle 4 `Lista prodotti` (30/10/2026)

Flusso `RINA - Search Items`. Modifica al solo **ramo `<= 3 caratteri`** del campo *Filter Query*
di `Lista prodotti 1..4`. Il ramo "codice" (parola con punto) e il ramo "parola lunga" **non
vengono toccati**.

## Espressione ORIGINALE (Lista prodotti 1; le altre differiscono solo per l'indice [1] [2] [3])

```
@concat('AVY0587_PurchaseNature eq Microsoft.Dynamics.DataEntities.AVYCR11352_ItemPurchaseNature''Hybrid'' and (', if(greater(length(body('Filtra_parole')), 0), if(contains(trim(string(body('Filtra_parole')[0])), '.'), concat('ProductNumber eq ''', trim(string(body('Filtra_parole')[0])), '*'''), if(lessOrEquals(length(trim(string(body('Filtra_parole')[0]))), 3), concat('ProductSearchName eq ''', trim(string(body('Filtra_parole')[0])), '*'' or ProductNumber eq ''', trim(string(body('Filtra_parole')[0])), '*'''), concat('ProductSearchName eq ''*', trim(string(body('Filtra_parole')[0])), '*'' or ProductNumber eq ''', trim(string(body('Filtra_parole')[0])), '*'''))), 'ProductNumber eq ''ZZNONEZZ'''), ')')
```

## La sola differenza introdotta

Nel ramo `<= 3 caratteri`:

| | |
|---|---|
| prima | `concat('ProductSearchName eq ''', trim(...)` |
| dopo  | `concat('ProductSearchName eq ''*', trim(...)` |

Cioe' un asterisco in piu' dopo `eq ''`: da *inizia con* a *contiene*, **solo per
`ProductSearchName`**. Il `ProductNumber eq 'parola*'` resta *inizia con*, che e' giusto per i
codici.

## Perche'

Caso `PS101002.0060`, domanda in portoghese, societa' PATR (30/10):
`ProductSearchName` = `Servizi Sanitari Locali (ISO,UNI, Linee Guida Farmindus...)`.
La parola `ISO` **e' nel campo** ma sta in mezzo, e il ramo corto la cercava solo in testa.
Le parole di tre lettere (`ISO`, `UNI`, `PC`, `IT`) sono spesso le piu' discriminanti.

## Rischio da misurare

Il *contiene* su parole corte e' la scansione piena del 2494: va confrontata la durata delle
`Lista prodotti` prima e dopo, sullo stesso caso.

Per annullare: rimettere l'espressione originale qui sopra in tutte e quattro, adattando l'indice.

---

## ESITO: RIPRISTINATO alle 17:52

La modifica e' rimasta pubblicata circa 23 minuti (17:29 - 17:52) e **non e' mai stata
validata**: nelle prove eseguite il modello non ha estratto nessuna parola di tre lettere, quindi
il ramo modificato non e' mai stato eseguito.

Ripristinate tutte e quattro al testo originale e ripubblicate. Verificate una per una prima di
pubblicare: **710 caratteri**, indici `[0] [1] [2] [3]`, **un** `ProductSearchName eq ''*`
(ramo lungo) e **un** `ProductSearchName eq ''` senza asterisco (ramo corto) — cioe' la
struttura di partenza.

**Il flusso e' esattamente com'era stamattina.** Nessuna modifica alle query e' rimasta in piedi
oggi, ne' questa ne' altre.

## Cosa ha prodotto comunque questo tentativo

La scoperta che le **parole chiave cambiano a ogni esecuzione** sulla stessa domanda
([[rina-keyword-non-deterministiche]]): e' il difetto a monte che spiega l'intermittenza
inseguita tutto il pomeriggio, e va chiuso prima di rimettere mano alle query.

## Se si riprende

1. prima rendere deterministica l'estrazione delle keyword;
2. poi riprovare il *contiene* sulle parole corte, **con una domanda che costringa a estrarne
   una di tre lettere**, altrimenti la prova non dice niente;
3. misurare durata delle quattro `Lista prodotti` e data limit prima e dopo.
