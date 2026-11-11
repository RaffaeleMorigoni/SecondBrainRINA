---
name: rina-knowledge-mapping-oldnew
description: Il mapping vecchio-nuovo MMD e le descrizioni estese sono knowledge file dell'agente, non ricerca per somiglianza
metadata:
  type: project
---

L'agente RINA ha due knowledge file caricati da Chiara D'Ambrosio a fine agosto 2026:
`Old_new_only_RINA.xlsx` (mapping 532 codici vecchi -> 405 MMD nuovi, univoco e senza buchi)
e `RINA_MD_Material_Codes_v1.xlsx` (682 codici con descrizioni estese in 6 lingue).

Quando l'utente parte da un codice vecchio la risposta passa da **Search sources /
Knowledge**, non dal flusso `Search Items`. Non e' somiglianza testuale: e' la tabella
ufficiale. La fonte originale e' il file cliente `old_new_with_matrix.xlsx`, foglio
`old new` - lo stesso da cui viene la matrice di contabilizzazione.

**Perche' conta:** e' facile analizzare il comportamento dell'agente guardando solo i
flussi e concludere che il mapping "non c'e'". Il 30/09/2026 ci sono cascato e ho
proposto al cliente di agganciare una tabella gia' agganciata da un mese.

**Come applicarlo:** prima di dire che manca un dato, guardare la scheda Knowledge
dell'agente, non solo Tools e i flussi. Due percorsi distinti: descrizione -> flusso,
codice vecchio -> knowledge. Se falliscono i casi da codice vecchio e non gli altri,
il sospetto va sulla knowledge. Vedi [[rina-dlp-knowledge-bloccata]].
