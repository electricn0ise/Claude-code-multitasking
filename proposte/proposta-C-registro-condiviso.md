# PROPOSTA C: il Changelog come registro condiviso (raffinamento del Sistema Memoria)

> **Stato: PROPOSTA preferita, non ancora applicata** (27/09/2026). Nasce dall'appunto di Matteo sulle Proposte [A](proposta-A-controlli-per-sessione.md) e [B](proposta-B-coordinatore-unico-scrittore.md): serve un sistema semplice e indipendente dal tipo di progetto, senza bloccare strumenti specifici. Non aggiunge database, campi, script o hook. Cambia soltanto **quando** si usa il Changelog e aggiunge una lettura prima di scrivere.

## Principio
Il mezzo di coordinamento tra le sessioni è **un unico registro condiviso: il 🕘 Changelog**. Le sessioni non devono conoscersi né sapere quale sia la "principale". Seguono tre regole, tutte legate a un solo concetto.

**Modifica reale** = qualunque scrittura sullo stato condiviso che un'altra sessione dovrebbe conoscere prima di toccare la stessa cosa. Esempi:
- commit o push su branch condivisi;
- rilascio o deploy;
- scrittura in HA (automazioni, dashboard, risorse, config);
- file definitivi.

Non lo sono le bozze e le prove in uno spazio proprio, che nessun altro usa.

1. **Scrivi nel registro quando fai la modifica, non a fine sessione.** Una riga: cosa, dove, riferimento (sha, `backup_id` o percorso) e sessione. Se il riferimento non c'è (per esempio codice non committato) lo si scrive esplicitamente.
2. **Leggi il registro prima di una modifica reale.** Le righe del sotto-progetto dall'ultima volta che hai letto. Se un'altra sessione ha cambiato ciò che stai per toccare, rileggi quella cosa dallo stato vero (git, HA, file) oppure chiedi.
3. **Una sessione ferma o gonfia non si riprende per fare modifiche reali.** Scrive un handoff nell'Attività e si riparte da una sessione nuova.

## Verifica sull'incidente della Dashboard 3D (27/09)
| Cosa è successo | Regola che l'avrebbe evitato |
|---|---|
| La v68 (sessione del ciclo solare) è stata registrata, ma la sessione ripresa non l'ha vista | 2: prima di rilasciare la v71 avrebbe letto la riga della v68 |
| Nessuna traccia di v64-v67, v69 e v70 | 1: una riga per ogni rilascio, subito |
| v71 rilasciata da codice non committato | 1: la riga avrebbe dovuto riportare uno sha, e la sua assenza si sarebbe vista |
| Il fix è andato a buon fine solo con una sessione nuova e un handoff | 3 |

## Modifiche al ⚙️ Core Protocol (testo da incollare nel synced block sulla pagina sorgente)

**DURANTE**. Sostituire:
> Non reinterrogare Notion per ogni file/commit. Tocca Notion a fine sessione, o per una Decisione stabile (append-only).

con:
> Non reinterrogare Notion per ogni file/commit. **Eccezione: prima di una modifica reale** (scrittura sullo stato condiviso che un'altra sessione dovrebbe conoscere: commit/push su branch condivisi, rilascio, scrittura in HA, file definitivi) **leggi il Changelog del sotto-progetto dall'ultima lettura**. Se un'altra sessione ha toccato ciò che stai per cambiare, rileggi lo stato vero (git/HA/file) o chiedi. **Subito dopo** la modifica reale scrivi la sua riga di Changelog. Per il resto tocca Notion a fine sessione, o per una Decisione stabile (append-only).

**FINE SESSIONE**. Sostituire:
> Changelog: 1 record solo per eventi consequenziali (transizione di stato, cambio versione, milestone/release, fix significativo, decisione). Append-only. […]

con:
> Changelog: già scritto a ogni modifica reale (vedi DURANTE), qui solo le voci mancanti per eventi consequenziali (transizione di stato, milestone, decisione). Ogni voce ha un riferimento (sha, `backup_id` o percorso). Se manca, va scritto esplicitamente ("codice non committato"). Append-only. […]

**Nuova riga in DURANTE** (regola 3):
> Sessione ferma da tempo o con il contesto oltre metà: **non riprenderla per una modifica reale**. Scrivi un handoff nell'Attività (`Handoff` = → Claude Code; `Contesto handoff` = 1-2 righe; documento completo nel corpo sotto `## Handoff <data>`: obiettivo misurabile, fatti con evidenze, regole, fasi con condizioni di STOP, fuori scope, "fatto quando") e riparti da una sessione nuova.

**AUTONOMO**. Invariato: "aggiungere una riga a Changelog" è già un'azione autonoma, solo comunicata.

## Costo
- Una riga di Changelog per ogni modifica reale.
- Una lettura del Changelog del sotto-progetto prima di farla.
- Zero token nel resto del tempo, e niente da configurare per progetto.

## Limiti dichiarati
- Nessun vincolo tecnico: il sistema si regge sul fatto che le sessioni seguano le regole. Sono tre e cadono in momenti naturali (prima e dopo una modifica).
- Il Changelog cresce di più, perché le serie di rilasci ravvicinati diventano più righe. È il prezzo della visibilità.
- Due modifiche nello stesso istante restano possibili (finestra stretta, accettata).

## Cosa resta delle Proposte A e B
Solo come strumenti opzionali, da aggiungere se servono per un progetto specifico: lo script `riallinea` per la parte git, il controllo prima del rilascio per la card, l'anteprima per ogni worker. Nessuno è necessario al sistema.
