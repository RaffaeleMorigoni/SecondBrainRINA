# ROLLBACK — Lista prodotti 1..4, filtro sul codice (29/10/2026)

Punto di ripristino preso prima di aggiungere il ramo "se la parola e' un codice, cerca solo su
ProductNumber". Per ripristinare: rimettere l'espressione nel campo *Filter Query* di ciascuna
azione, poi `Save draft` → ricaricare la pagina → `Publish`.

Le quattro azioni sono **identiche**: cambia solo l'indice `N` (0, 1, 2, 3) e il confronto
`greater(length(body('Filtra_parole')), N)`. Verificate una per una il 29/10.

| azione | indice | greater(..., N) |
|---|---|---|
| Lista_prodotti_1 | `[0]` | 0 |
| Lista_prodotti_2 | `[1]` | 1 |
| Lista_prodotti_3 | `[2]` | 2 |
| Lista_prodotti_4 | `[3]` | 3 |

Altri parametri, invariati: `table` = ProductsV2, `$top` = 40, `$select` = ProductNumber,
`runAfter` = Filtra_parole / Succeeded.

## $filter ORIGINALE (qui con indice 0 — sostituire `[0]` e `, 0)` per le altre)

```
@concat('AVY0587_PurchaseNature eq Microsoft.Dynamics.DataEntities.AVYCR11352_ItemPurchaseNature', decodeUriComponent('%27'), 'Hybrid', decodeUriComponent('%27'), ' and (', if(greater(length(body('Filtra_parole')), 0), if(lessOrEquals(length(trim(string(body('Filtra_parole')[0]))), 3), concat('ProductSearchName eq ', decodeUriComponent('%27'), trim(string(body('Filtra_parole')[0])), '*', decodeUriComponent('%27'), ' or ProductNumber eq ', decodeUriComponent('%27'), trim(string(body('Filtra_parole')[0])), '*', decodeUriComponent('%27')), concat('ProductSearchName eq ', decodeUriComponent('%27'), '*', trim(string(body('Filtra_parole')[0])), '*', decodeUriComponent('%27'), ' or ProductNumber eq ', decodeUriComponent('%27'), trim(string(body('Filtra_parole')[0])), '*', decodeUriComponent('%27'))), concat('ProductNumber eq ', decodeUriComponent('%27'), '#none#', decodeUriComponent('%27'))), ')')
```

## $filter NUOVO (indice 0)

```
@concat('AVY0587_PurchaseNature eq Microsoft.Dynamics.DataEntities.AVYCR11352_ItemPurchaseNature''Hybrid'' and (', if(greater(length(body('Filtra_parole')), 0), if(contains(trim(string(body('Filtra_parole')[0])), '.'), concat('ProductNumber eq ''', trim(string(body('Filtra_parole')[0])), '*'''), if(lessOrEquals(length(trim(string(body('Filtra_parole')[0]))), 3), concat('ProductSearchName eq ''', trim(string(body('Filtra_parole')[0])), '*'' or ProductNumber eq ''', trim(string(body('Filtra_parole')[0])), '*'''), concat('ProductSearchName eq ''*', trim(string(body('Filtra_parole')[0])), '*'' or ProductNumber eq ''', trim(string(body('Filtra_parole')[0])), '*'''))), 'ProductNumber eq ''ZZNONEZZ'''), ')')
```

### Cosa cambia, in tre punti

1. **Ramo nuovo, il primo test**: se la parola **contiene un punto** la si tratta come codice e si
   interroga **solo `ProductNumber`**. Sparisce il `ProductSearchName eq '*codice*'`, che e' un
   *contiene* con wildcard in testa e costringe alla scansione piena della tabella: era la causa
   del `TimedOut` del 29/10 alle 14:27.
2. **`#none#` → `ZZNONEZZ`**: debito noto dal 05/10, questo era uno dei quattro campi rimasti
   (vedi memoria `rina-sentinella-none-filtri`). Si attiva solo quando la parola non esiste,
   quindi non interferisce con il punto 1.
3. **`decodeUriComponent('%27')` → apostrofo raddoppiato**: stesso risultato, espressione da ~900
   a ~640 caratteri. Serve anche a poterla digitare senza che il campo vada in timeout.

La ricerca per descrizione **non e' toccata**: i rami "parola corta" e "parola lunga" restano
identici, wildcard in testa compreso.
