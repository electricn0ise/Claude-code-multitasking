# PROPOSTA C: il Changelog come registro condiviso (raffinamento del Sistema Memoria)

> **Stato: PROPOSTA preferita, non ancora applicata** (27/09/2026). Nasce dall'appunto di Matteo sulle Proposte [A](proposta-A-controlli-per-sessione.md) e [B](proposta-B-coordinatore-unico-scrittore.md): serve un sistema semplice e indipendente dal tipo di progetto, senza bloccare strumenti specifici. Non aggiunge database, script o hook. Cambia soltanto **quando** si usa il Changelog, aggiunge una lettura prima di scrivere e un solo campo al Changelog (`Riferimento`).

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
| La sessione del ciclo solare aveva già registrato ogni rilascio (v64-v68) quasi in tempo reale: la regola 1 era rispettata di fatto, mancava la regola 2 dall'altra parte. Nessuna traccia di v69 e v70 (forse mai rilasciate) | 1 resa esplicita per tutte le sessioni; 2 |
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
> Changelog: già scritto a ogni modifica reale (vedi DURANTE), qui solo le voci mancanti per eventi consequenziali (transizione di stato, milestone, decisione). Ogni voce ha il campo `Riferimento` compilato (sha, `backup_id` o percorso; se manca, "non committato"). Append-only. […]

**Nuova riga in DURANTE** (regola 3):
> Sessione ferma da tempo o con il contesto oltre metà: **non riprenderla per una modifica reale**. Scrivi un handoff nell'Attività (`Handoff` = → Claude Code; `Contesto handoff` = 1-2 righe; documento completo nel corpo sotto `## Handoff <data>`: obiettivo misurabile, fatti con evidenze, regole, fasi con condizioni di STOP, fuori scope, "fatto quando") e riparti da una sessione nuova.

**AUTONOMO**. Invariato: "aggiungere una riga a Changelog" è già un'azione autonoma, solo comunicata.

## Modifiche al 🕘 Changelog
I campi attuali coprono quasi tutto:

| Serve alla Proposta C | Campo | Stato |
|---|---|---|
| Cosa è cambiato | `Nome` | invariato |
| Sotto-progetto e progetto (per filtrare in lettura) | `Sotto-progetto`, `Progetto` | invariati |
| Quando, con l'ora (lettura "dall'ultima volta") | `createdTime` (automatico) | invariato; `Data` ha solo il giorno e resta com'è |
| Quale sessione | `Sessione` | invariato |
| Tipo di evento | `Tipo` | invariato |
| Riferimento (sha, `backup_id`, percorso o "non committato") | **`Riferimento` (testo), nuovo** | modifica di schema, da approvare |

**Perché `Riferimento` è un campo e non una riga nel corpo:** la regola 2 legge il registro con una query, e una query restituisce le proprietà ma **non il corpo delle pagine**. Nel corpo il riferimento sarebbe invisibile alla lettura. `Backup` resta com'è: ha un significato preciso (tarball più `Rollback target`) e lo usa il controllo di integrità. Per una scrittura in HA con backup si compilano tutti e due.

**Struttura:** nessun cambiamento. Una riga per ogni modifica reale, append-only.

**Formato di una riga** (da mettere nel Core Protocol, nessun template obbligatorio):
- `Nome`: "<cosa> (<versione, se c'è>)", per esempio "Dashboard 3D: onde Wi-Fi più lente (v71)";
- `Riferimento`: sempre compilato, per esempio `sweet-home-3d-casa@201743b`, `backup 69afda2a`, `/config/www/dashboard3d/card.js`, oppure "non committato";
- `Tipo`, `Sotto-progetto`, `Sessione` come oggi; corpo facoltativo.

Un template di pagina Notion è utile solo per le righe scritte a mano da Matteo, quindi è facoltativo.

**Vista "Registro recente"** (non tocca lo schema): ordinata per `createdTime` dal più recente, raggruppata per `Sotto-progetto`, con le colonne `Nome`, `Riferimento`, `Sessione` e `createdTime`. Serve a Matteo per vedere a colpo d'occhio chi ha cambiato cosa.

**Query della regola 2** (lettura prima di una modifica reale):
```sql
SELECT "Nome", "Tipo", "Riferimento", "Sessione", createdTime
FROM "collection://f1c27200-7819-415d-a64c-da6ade9c4ebc"
WHERE "Sotto-progetto" LIKE '%<id del sotto-progetto>%'
  AND datetime(createdTime) > datetime('<ultima lettura di questa sessione>')
ORDER BY createdTime DESC
LIMIT 10
```
Il confronto passa **sempre** da `datetime(...)`. `createdTime` è salvato come `2026-09-26 16:08:25Z`: un confronto tra stringhe con un valore scritto `2026-09-26T…` scarta in silenzio le righe di quel giorno, perché lo spazio viene prima della `T` (errore fatto davvero durante l'analisi dell'incidente). La query è stata provata sul Changelog reale con il sotto-progetto Dashboard 3D e una finestra dal 26/09 13:32: restituisce le righe v64-v68, v71 e la Verifica del riallineamento, ognuna con la sua sessione.
"Ultima lettura" è l'ora dell'ultima query o del fetch al boot fatti in questa sessione. Se la sessione non la conosce, si usano le ultime 48 ore. Le righe con `Sessione` diversa dalla propria sono quelle da guardare.

**🩺 Integrity Check** (controllo mensile): alla verifica esistente sul campo `Backup` si aggiunge "ogni record creato dopo l'adozione della Proposta C ha `Riferimento` non vuoto". È il modo per accorgersi a posteriori se la regola 1 viene saltata.

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
