# PROPOSTA A: coordinare sessioni parallele nei progetti grandi (senza sessione principale)

> **Stato: PROPOSTA, non approvata e non implementata** (27/09/2026). Messa da parte per confrontarla con altre possibilità prima di decidere (vedi [Proposta B](proposta-B-coordinatore-unico-scrittore.md)). Nessun deliverable descritto qui è stato creato.

## Context
Matteo divide i progetti grandi in molte sessioni, per evitare il context bloat, e la gerarchia tra sessioni è fluida: nessuna sessione è stabilmente "principale". L'incidente della Dashboard 3D (27/09), ricostruito su GitHub, Changelog Notion e metadati delle sessioni, mostra dove questo si rompe. Ogni pezzo del piano risponde a un guasto osservato; ciò che non è legato all'incidente è escluso.

| Guasto osservato | Pezzo | Costo fuori da quel momento |
|---|---|---|
| Il fix è arrivato a convergenza solo quando una **sessione nuova** ha ricevuto un **handoff autosufficiente**. La sessione "principale" (546k token di contesto) era ormai ferma a uno stato vecchio. | **1. Handoff come formato standard e "riparti da fresco"** | zero |
| La sessione ripresa non sapeva della v68, che era **già nel Changelog**: il protocollo le vietava di rileggere Notion a metà sessione. | **2. `/riallinea` alla ripresa** | zero |
| v71 rilasciata da codice non committato; nessun record per v64-v67, v69, v70; rilasci da un branch verso l'unica produzione. | **3. Controllo prima di scrivere in produzione** | zero |
| Due sessioni vive sullo stesso checkout. | **4. Una sessione attiva per checkout** | zero |

**Verifica di topologia:** ogni pezzo gira per sessione e non dipende da quale sessione sia "principale".

**Valutati, non ora:** Projects (beta, e i thread locali non hanno worktree né memoria), agent teams (sperimentali), lock o registri di sessioni, hook su ogni prompt (troppo rumore).

## 1. Handoff standard + "riparti da fresco" (il pezzo principale)
- **Skill `handoff`**: produce il documento nel formato che ha funzionato col fix v71:
  - obiettivo misurabile;
  - fatti accertati, ognuno con la sua evidenza (sha, id di backup, sessione);
  - regole e divieti;
  - fasi, ognuna con esito atteso e condizioni di STOP;
  - fuori scope;
  - lista per dire che il lavoro è finito.
  **Dove si scrive (su Notion, campi già esistenti, nessuna modifica di schema):**
  - `Handoff` (selezione) = `→ Claude Code`, oppure `→ claude.ai` se serve una decisione di Matteo;
  - `Contesto handoff` (testo breve): 1-2 righe con l'obiettivo e "documento completo nella pagina, sezione Handoff <data>";
  - **il documento completo va nel corpo della pagina dell'Attività** sotto `## Handoff <data>`, perché un campo di testo è troppo corto per titoli, blocchi di codice e liste di controllo;
  - se l'Attività non esiste, la skill la **propone** (il protocollo richiede di proporre prima una nuova Attività) e intanto salva il documento in un file locale.
  - A chiusura: il protocollo già prevede di svuotare `Handoff` e spostare il contesto in una nota nella pagina. La sezione resta nel corpo come storia.
  - La sessione nuova parte con "riprendi l'Attività X": BOOT → fetch dell'Attività → lettura del corpo.
- **Regola**: per lavoro consequenziale (rilascio, merge, riconciliazione) su una sessione ferma da tempo o con il contesto oltre metà, la sessione vecchia **scrive l'handoff** e il lavoro **riparte in una sessione nuova**. Risponde anche all'obiettivo di partenza: meno context bloat, e continuità che non dipende dalla memoria di una singola sessione.

## 2. `/riallinea` (a richiesta, alla ripresa)
- **`riallinea.sh`**, parte git deterministica e con output limitato:
  - branch corrente, `status`;
  - `HEAD..main` e `main..HEAD`;
  - branch non mergiati: al massimo 5, ognuno con ultimo commit, numero di commit e numero di file **nuovi** o modificati;
  - worktree presenti.
- **Repo fuori dalla cartella di sessione (il caso di Matteo):** lo script prende il percorso del repo come argomento e usa `git -C "<repo>"`, quindi non dipende dalla cartella della sessione. La skill decide quale repo guardare in quest'ordine:
  1. percorso passato esplicitamente (`/riallinea "C:\Users\Elect\Progetti\Sweet Home 3D"`);
  2. i repo dei file che la sessione ha già letto o modificato (`git -C <cartella del file> rev-parse --show-toplevel`), uno o più;
  3. altrimenti chiede a Matteo.

  Aggiungere un campo `Repo` ai Sotto-progetti renderebbe il passo automatico, ma è una modifica di schema: solo come proposta, non inclusa. Nota di installazione: aggiungere i repo esterni a `permissions.additionalDirectories` (o usare `/add-dir`), e mettere lo script in allowlist, per evitare richieste di permesso a ogni lancio.
- **Skill `riallinea`**:
  - lancia lo script;
  - interroga il **Changelog** e le **Decisioni** (relazione `Sotto-progetto`, verificata) **del sotto-progetto** dall'ultima lettura di Notion in questa sessione (il timestamp "as of" del fetch al boot) oppure, in mancanza, nelle ultime 48 h, con al massimo 10 righe per database;
  - legge la versione live, se il repo ha un timbro di build;
  - chiude con 3-5 righe di raccomandazione ("fai merge di `main`", "rileggi X", "non rilasciare", "passa a una sessione nuova").

## 3. Controllo prima di scrivere in produzione
È la regola che il protocollo HA ha già ("discover/verify live → plan → change"), applicata anche ai **file rilasciati da un repo**:
- si rilascia solo da un commit pulito e pushato;
- timbro `BUILD v=<n> branch=<b> commit=<sha>` sulla copia caricata;
- prima di sovrascrivere: `git fetch`, poi `--is-ancestor <sha live> HEAD`. STOP se non è un antenato, se lo sha è sconosciuto o se manca il timbro;
- `?v=` = valore live + 1;
- **ogni scrittura in produzione ha il suo record nel Changelog**.

Lo script generico è `release-guard.sh`. L'applicazione alla Dashboard 3D (README della card e script nel repo) passa da un handoff a una sessione locale, perché da qui su `sweet-home-3d-casa` ho solo lettura.

## 4. Una sessione attiva per checkout (default)
- Prima di lavorare su un repo, `/riallinea` mostra branch e stato. Se un'altra sessione è attiva sulla stessa cartella, ci si ferma e si chiede a Matteo.
- Il worktree (opzione "worktree" dell'app desktop, oppure `git worktree add`) serve solo quando due sessioni devono davvero modificare lo stesso repo in parallelo. Nel layout di Matteo ha attrito: la cartella di sessione non è il repo, il BOOT non riconosce il progetto, e `CASA.sh3d` viene copiato in ogni worktree (va trattato in sola lettura).

## Progetti senza repo (es. automazioni HA)
Il protocollo HA prevede di aggiornare le automazioni con **"remove + recreate"**, cioè una sovrascrittura completa: è lo stesso meccanismo della v71. La sessione A legge l'automazione X, la sessione B la modifica, A la ricrea dalla sua copia vecchia e **annulla in silenzio** il lavoro di B.

- **Regola nuova (la più importante lato HA):** subito **prima** di ogni scrittura si rilegge la config live e la si confronta con la versione su cui si è basata la modifica. Se sono diverse, ci si ferma e si mostra il diff. Si usa un diff e non un hash, perché HA può riordinare le chiavi o aggiungere valori di default, e il diff si legge. Il "discover/verify live" all'inizio del flusso non basta: in una sessione lunga può essere avvenuto ore prima.

| Pezzo | Con un repo | Senza repo (HA) |
|---|---|---|
| 1. Handoff e ripartenza da fresco | uguale | uguale (vive su Notion) |
| 2. `/riallinea` | git + Changelog + Decisioni | solo Changelog + Decisioni |
| 3. Controllo prima di scrivere | timbro + `--is-ancestor` | rilettura live + diff subito prima della scrittura |
| 4. Isolamento | una sessione per checkout | una sessione alla volta per **label HA** (= sotto-progetto, già nel protocollo). Label diverse possono lavorare in parallelo; un'entità usata da due sotto-progetti → STOP e si chiede |
| Storia | git + Changelog | **solo Changelog**, quindi "un record per ogni scrittura in produzione" è ancora più importante |

Da verificare sul PC, non ancora nel design: se `ha_manage_backup scope=edits` (visto nel Changelog) fornisce una storia delle modifiche per automazione, e se i timestamp del registro HA servono a qualcosa.

## Cosa non copre (nessuno di questi punti blocca il piano)
- Decisioni di design divergenti che non arrivano in produzione: copertura parziale, perché `/riallinea` mostra le Decisioni recenti.
- `/riallinea` è manuale e ci si può dimenticare di lanciarlo: la rete di sicurezza è il controllo prima della scrittura, che fa parte della procedura di scrittura stessa.
- Il protocollo è fatto di istruzioni, non di vincoli: per i file c'è uno script, per HA c'è solo la regola.
- Due scritture nello stesso istante: finestra stretta, accettata.

## Deliverable
Nel repo `Claude-code-multitasking`, branch `claude/gracious-hopper-2yhywn`, con commit e push:
- `skills/handoff/SKILL.md`
- `skills/riallinea/SKILL.md` + `skills/riallinea/riallinea.sh`
- `tools/release-guard.sh`
- `tests/`: replay dell'incidente (vedi Verification)
- `protocollo/emendamenti.md`: testo pronto da incollare nel Core Protocol (eccezione "rileggi Notion alla ripresa e prima di un rilascio", regola "riparti da fresco", una sessione per checkout) e nel protocollo HA (verifica live anche per i file rilasciati; **rilettura live + diff subito prima di ogni scrittura, soprattutto con remove + recreate**; una sessione per label; un record Changelog per ogni scrittura in produzione)
- `handoff/sweet-home-regole-rilascio.md`: handoff per una sessione locale che applica il punto 3 al repo della dashboard
- `README.md`: indice e installazione (copia in `~/.claude/skills/`)

Notion resta in sola lettura: gli emendamenti sono testo da incollare, e nessun campo nuovo.

## Verification (qui, sul clone di `sweet-home-3d-casa`)
Si ricostruisce lo stato precedente al fix: `main@58196ec`, `sole@919c279`.
- `riallinea.sh` lanciato da una cartella **diversa** dal repo (come nel layout di Matteo), con il percorso del clone come argomento, su `main` → riporta `sole` non mergiato (24 commit, 15 file nuovi e 3 modificati), con output entro i limiti.
- `release-guard.sh`:
  - live timbrato `commit=919c279` contro HEAD=`58196ec` → **STOP**;
  - sha sconosciuto → STOP;
  - senza timbro → STOP;
  - live `58196ec` contro HEAD=`919c279` → **PASS**.
- Query Changelog dello skill `riallinea` con finestra dal 26/09 → restituisce i record v68 e v71 (query in sola lettura qui).
- `bash -n` e test in `tests/` verdi.
- **Parte HA (solo sul campo):** due sessioni locali. B modifica l'automazione X; A prova a fare remove + recreate di X → atteso STOP con il diff.
