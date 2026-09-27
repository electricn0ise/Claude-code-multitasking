# PROPOSTA C: il Changelog come registro condiviso (raffinamento del Sistema Memoria)

> **Stato: PROPOSTA preferita, non ancora applicata** (27/09/2026, revisione 4). Nasce dall'appunto di Matteo sulle Proposte [A](proposta-A-controlli-per-sessione.md) e [B](proposta-B-coordinatore-unico-scrittore.md): serve un sistema semplice e indipendente dal tipo di progetto, senza bloccare strumenti specifici.
>
> Non aggiunge database e non blocca strumenti. Cambia **quando** si scrive il 🕘 Changelog, aggiunge alle 📋 Attività la **presa in carico** (idea di Matteo), aggiunge due letture prima di scrivere e un **hook di avviso** facoltativo che informa le sessioni già avviate. Le correzioni rispetto alle stesure precedenti sono in fondo, nella sezione "Revisioni".

## Principio
Le sessioni non devono conoscersi né sapere quale sia la "principale". Si coordinano attraverso tre fonti, ognuna con il suo proprietario, come vuole la matrice di ownership del Core Protocol:

| Domanda | Fonte |
|---|---|
| Cosa è cambiato? | 🕘 **Changelog**, il registro condiviso (append-only) |
| Chi sta lavorando su cosa, e dove? | 📋 **Attività**, con la presa in carico |
| Cosa c'è davvero adesso? | **Stato vero**: git, HA, file |

Prima di scrivere si guardano tutte e tre. Una sessione già avviata viene **avvisata** dall'hook quando qualcosa cambia.

### Definizioni
**Modifica reale.** Una scrittura su qualcosa che altre sessioni, o Matteo, usano o leggono:
- push o merge su un branch condiviso (una riga per push, non per commit);
- rilascio o deploy;
- scrittura in HA (automazioni, dashboard, risorse, config, `/config/www`);
- file definitivi (per esempio `CASA.sh3d`);
- cambio di branch in un checkout usato anche da altri.

Non sono modifiche reali:
- modificare file o committare sul proprio branch;
- lavorare in un worktree o in una copia propria;
- le anteprime e le entità di prova proprie;
- le scritture su Notion.

**Spazio condiviso.** Una cartella o risorsa che più sessioni possono toccare: il checkout principale di un repo, una risorsa in copia unica (`CASA.sh3d`), la produzione HA. Uno spazio proprio (worktree, copia, anteprima) non è condiviso.

**Presa in carico.** Un'Attività con `Stato` = *In corso*, il campo `Sessione` compilato (l'URL della sessione, oppure "Matteo" se ci lavora lui a mano) e, se tocca uno spazio condiviso, il campo `Dove` compilato. `Dove` indica l'**identità dello spazio, senza il branch**: per esempio "checkout principale sweet-home-3d-casa", "HA produzione: card dashboard3d", "CASA.sh3d". Stesso spazio vuol dire collisione, qualunque sia il branch.
- **Serve** solo per lavorare in uno spazio condiviso, o per una serie di modifiche reali alla stessa cosa (per esempio le prove ripetute sul tablet). Il lavoro in uno spazio proprio non la richiede.
- **Si prende:** si scrive `Sessione` (e `Dove`), poi si rilegge l'Attività, come vuole la regola SCRITTURA ("read-back"). Se c'è un altro valore, ci si ritira.
- **Si rilascia:** `Stato` passa ad altro e `Sessione` si svuota. Anche un handoff è un rilascio: `Handoff` = → Claude Code e `Sessione` vuota. La sessione nuova prende in carico scrivendo la propria.
- **Righe esistenti:** le Attività già *In corso* senza `Sessione` contano come non prese in carico.

**Base non affidabile.** La sessione non può più fidarsi di ciò che ha in contesto. Succede in tre casi:
- Matteo lo dice;
- dopo una compaction, la regola 2 mostra cambiamenti altrui troppo estesi per integrarli rileggendo pochi file;
- la sessione è stata avviata prima dell'adozione di questo sistema.

## Le tre regole

**1. Scrivi nel registro subito dopo ogni modifica reale.** Una riga con:
- `Nome`: cosa, e la versione se c'è;
- `Riferimento`: sha, `backup_id` o percorso, altrimenti "non committato";
- `Sotto-progetto`: tutti quelli toccati, oppure vuoto con solo `Progetto` se la modifica è di piattaforma;
- `Progetto`, `Tipo`, `Sessione`.

Per una **serie** alla stessa cosa, sotto presa in carico, basta una riga alla fine, con il riferimento finale. Se la riga non si riesce a scrivere (per esempio Notion è irraggiungibile), niente altre modifiche reali finché non ci si riesce, salvo decisione di Matteo; si tiene l'elenco delle righe mancanti e si registrano appena possibile.

**2. All'inizio di ogni compito e prima di una modifica reale, leggi.**
- **(a) Registro e Attività.** Si leggono le viste *Registro recente* e *Attività in corso* (vedi sotto). Il *Registro recente* si legge dall'alto fino alla prima riga già vista; se non ricordi nessuna riga vista (sessione nuova o dopo una compaction), due pagine. Contano le righe non tue del tuo sotto-progetto, o di piattaforma del tuo progetto. In particolare:
  - **presa in carico di un'altra sessione** sulla stessa cosa, o con lo stesso `Dove` → worktree proprio, oppure chiedi a Matteo;
  - **un'Attività che avevi preso in carico non porta più la tua `Sessione`** (vuota o diversa) → non sei più il proprietario: niente modifiche reali, ti fermi;
  - **una riga altrui "non committato"** → la produzione è avanti rispetto a git: la integri prima di qualunque rilascio.
- **(b) Stato vero di ciò che stai per sovrascrivere.** Rileggi il file live, la config, la punta del branch remoto (`git fetch`) e, in uno spazio condiviso, controlla se ci sono modifiche non tue (`git status`). È la difesa contro chi scrive fuori dal registro: Matteo dall'interfaccia, aggiornamenti automatici, righe dimenticate.

Se (a) o (b) mostrano cambiamenti non tuoi, **prima integri e poi scrivi** (merge, rilettura, adattamento), oppure chiedi a Matteo.

Eccezioni, per non ripetere letture inutili:
- se l'hook ha appena riportato le novità, la (a) a inizio compito è già fatta;
- per modifiche reali consecutive alla stessa cosa, nello stesso turno di lavoro, la (a) si può saltare; la (b) resta sempre;
- dopo una compaction si rifanno (a) e (b) per intero.

**3. Con una base non affidabile non si fanno modifiche reali: si scrive un handoff e si riparte.** L'handoff va nell'Attività:
- `Handoff` = → Claude Code e `Sessione` vuota;
- `Contesto handoff`: 1-2 righe;
- documento completo nel corpo sotto `## Handoff <data>`: obiettivo misurabile, fatti con evidenze, regole, fasi con condizioni di STOP, fuori scope, "fatto quando".

Chi ha scritto un handoff non fa più modifiche reali.

## L'avviso alle sessioni già avviate
Una sessione non può accorgersi da sola che qualcosa è cambiato: può solo guardare, o essere avvisata. Si distinguono due casi.

**Sessione ferma, che riceve un messaggio di Matteo (il caso della dashboard): hook `hook-registro`.** È un hook `UserPromptSubmit` sul PC, a livello utente, in Python.
- A ogni messaggio interroga via API Notion, in sola lettura:
  - il Changelog: righe create dopo l'ultimo controllo di questa sessione;
  - le Attività: righe modificate dopo l'ultimo controllo.
- Tiene solo le righe del **progetto della cartella della sessione**. L'abbinamento cartella → `Repo` è lo stesso del BOOT; senza abbinamento tiene tutti i progetti.
- Esclude le righe **scritte da questa sessione**. Le riconosce grazie a un secondo hook `PostToolUse`, silenzioso, sullo strumento Notion di creazione delle pagine: annota gli id delle pagine create dalla sessione.
- **Se non c'è niente di nuovo non stampa niente**, anche dopo una pausa di giorni. Se c'è qualcosa, stampa al massimo tre righe, poi "+N". Per esempio: *"[registro] da altre sessioni: Dashboard 3D v68 (27/09 00:13, rif. non committato); Attività 'Ciclo solare' presa in carico da un'altra sessione"*.
- **Primo controllo di una sessione:** il punto di partenza è il primo timestamp del file di transcript della sessione (`transcript_path`). Così anche una sessione avviata prima dell'installazione dell'hook vede tutto ciò che è cambiato da quando è partita.
- **Se Notion non risponde** (timeout di 2-3 secondi): non blocca mai il messaggio. Se dall'ultimo messaggio sono passati più di 30 minuti stampa *"[pausa di X h: registro non raggiungibile, rileggilo (regola 2a)]"*, altrimenti niente.
- **Token Notion:** un'integrazione **in sola lettura**, condivisa solo con Changelog, Attività e Progetti, con il token in una variabile d'ambiente del PC.

**Sessione al lavoro da sola a lungo: campanello tra sessioni.** Chi fa una modifica reale guarda le *Attività in corso* dello stesso sotto-progetto o dello stesso `Dove`. Alle sessioni che le hanno in carico manda un `SendMessage` di una riga (*"ho rilasciato la v68 della card, riga nel registro"*), che arriva al loro prossimo passo di lavoro. È solo un campanello: se non arriva, resta la regola 2 prima di scrivere.

**L'hook è facoltativo:** senza, il sistema funziona con le sole regole, perché la 2(a) a inizio compito fa lo stesso lavoro, solo spendendo token. Se lo script si rompe si torna a quella situazione, e non si rompe nient'altro.

## Scenari di prova (percorsi sulla carta)
| Scenario | Cosa succede | Esito |
|---|---|---|
| **Due sessioni vive nello stesso checkout** (l'incidente del 26-27/09) | La sessione del ciclo solare prende in carico "Ciclo solare" con `Dove` = "checkout principale sweet-home-3d-casa". L'altra, qualunque branch usi, lo vede con l'hook o con la 2(a): worktree proprio, oppure chiede. Se la presa in carico manca, prima di scrivere la 2(b) trova con `git status` modifiche non sue e si ferma. | regge |
| **Matteo torna sulla sessione vecchia e scrive "continua"** (il sintomo segnalato) | L'hook stampa le righe v64-v68 dell'altra sessione e la presa in carico del ciclo solare prima che la sessione ragioni. Senza hook, se ne accorge prima della prossima modifica reale (2a). | regge (con l'hook subito, senza hook alla scrittura) |
| **Rilascio da codice non committato** (v71) | La riga porta "non committato". Chi legge sa che la produzione è avanti rispetto a git e la integra prima di un rilascio; il controllo di integrità lo elenca come debito. | regge |
| **Sessione vecchia che continua dopo un handoff scritto da un'altra** (il caso reale del 27/09) | L'handoff ha svuotato `Sessione` e la sessione nuova ha scritto la propria. Alla vecchia l'hook segnala al primo messaggio "Attività presa in carico da un'altra sessione", e la 2(a) prima del commit la ferma. | regge per le sessioni che avevano preso in carico; quelle avviate prima dell'adozione ricadono nella regola 3 |
| **Matteo modifica un'automazione dall'interfaccia**, poi una sessione fa remove + recreate | Nel registro non c'è niente, ma la 2(b) rilegge la config live subito prima di scrivere, vede la differenza e integra o chiede. | regge |
| **Sessione ripresa dopo tre giorni** | L'hook riporta tutto ciò che è cambiato da allora (al massimo tre righe, poi "+N" e l'invito a leggere la vista). | regge |
| **Ciclo lungo di prove sul tablet** | Presa in carico con `Dove` = "HA produzione: card", 2(b) prima di ogni rilascio, una riga di Changelog alla fine. Le altre sessioni vedono la presa in carico. | regge, con una riga invece di dieci |
| **Sessione al lavoro da ore mentre un'altra rilascia** | Il campanello arriva al suo prossimo passo. Se non arriva, la 2(a) e la 2(b) prima della sua prossima modifica reale. | regge, con un possibile ritardo fino alla prossima scrittura |
| **Presa in carico appesa** (la sessione è morta) | L'Attività resta *In corso* con una `Sessione` inattiva. Chi la trova chiede a Matteo; il controllo mensile la segnala. | regge |
| **Notion irraggiungibile** | L'hook tace o dà l'avviso di pausa; stop alle modifiche reali finché la riga non si può scrivere, salvo decisione di Matteo. | regge |

## Modifiche al ⚙️ Core Protocol (sempre caricato: aggiunta corta)
Nel Core Protocol vanno solo quattro righe. Definizioni e dettagli vanno in un record di 📐 Protocolli letto **solo quando serve**.

**DURANTE**. Sostituire:
> Non reinterrogare Notion per ogni file/commit. Tocca Notion a fine sessione, o per una Decisione stabile (append-only).

con:
> Non reinterrogare Notion per ogni file/commit, **salvo il coordinamento tra sessioni** (regole complete nel Protocollo *Registro condiviso*, da leggere prima della prima modifica reale o presa in carico della sessione): (1) subito dopo ogni modifica reale, una riga di Changelog con `Riferimento`; (2) a inizio compito e prima di una modifica reale, leggi le viste *Registro recente* e *Attività in corso* e rileggi dallo stato vero ciò che sovrascrivi; per uno spazio condiviso prendi in carico l'Attività (`Sessione`, `Dove`); (3) base non affidabile → handoff e sessione nuova. Per il resto tocca Notion a fine sessione, o per una Decisione stabile (append-only).

**FINE SESSIONE**. Nella riga del Changelog, sostituire "1 record solo per eventi consequenziali (…)" con:
> le modifiche reali sono già registrate (DURANTE); qui solo gli eventi consequenziali che non lo sono (transizione di stato, milestone, decisione). Rilascia le prese in carico (`Sessione` vuota) o lasciale esplicitamente in un handoff. Ogni riga ha `Riferimento`.

**MATRICE DI OWNERSHIP**. Sostituire "Stato di esecuzione di una fetta di lavoro → **Attività**" con:
> Stato di esecuzione di una fetta di lavoro → **Attività** (inclusa la presa in carico: `Sessione`, `Dove`)

**AUTONOMO vs DA PROPORRE.** Prendere e rilasciare in carico un'Attività è **autonomo**, solo comunicato. Raccomandazione, da approvare: anche **aprire un'Attività** diventa autonomo quando sta dentro un sotto-progetto esistente e serve a prendere in carico il lavoro che la sessione sta per fare. Altrimenti le sessioni salterebbero la presa in carico per evitare il passaggio di conferma.

**Nuovo record in 📐 Protocolli: "Registro condiviso".** Contiene le sezioni "Definizioni", "Le tre regole" e "L'avviso alle sessioni già avviate" di questo documento.

## Modifiche allo schema (da approvare)
**🕘 Changelog**
| Campo | Stato |
|---|---|
| `Nome`, `Progetto`, `Sotto-progetto` (anche più di uno), `Tipo`, `Sessione`, `Backup`, `Rollback target`, `Data` | invariati |
| **`Riferimento`** (testo): sha, `backup_id`, percorso o "non committato" | nuovo |
| **`Creato`** (*created time*, automatico, compilato anche per le righe esistenti) | nuovo, serve a ordinare la vista |

`Riferimento` è un campo e non una riga nel corpo perché letture via vista, query e API restituiscono le proprietà, **non il corpo delle pagine**. `Backup` resta com'è: per una scrittura in HA con backup si compilano tutti e due.

**📋 Attività**
| Campo | Stato |
|---|---|
| `Stato` (opzione *In corso* già esistente), `Handoff`, `Contesto handoff`, `Prossimo passo`, `Sotto-progetto`, `Progetto`, … | invariati |
| **`Sessione`** (testo): chi ha preso in carico l'Attività (URL della sessione, oppure "Matteo") | nuovo |
| **`Dove`** (testo): identità dello spazio condiviso, senza branch | nuovo |

**Viste** (lette in modalità vista):
- ***Registro recente*** (Changelog): ordinata per `Creato` dal più recente, senza filtri; colonne `Nome`, `Tipo`, `Riferimento`, `Sotto-progetto`, `Progetto`, `Sessione`, `Creato`.
- ***Attività in corso*** (Attività): filtro `Stato` = *In corso*; colonne `Nome`, `Sotto-progetto`, `Sessione`, `Dove`, `Prossimo passo`.

Tutte e due servono anche a Matteo per vedere a colpo d'occhio chi fa cosa.

**Perché le sessioni leggono tramite vista e non con query SQL.** Il workspace non è su piano Business: `query_data_sources` risulta *available_with_limit*, e la quota delle query si è già esaurita una volta (🧠 Sistema Memoria — Design & Storia). L'hook usa invece l'API REST pubblica di Notion con il proprio token, che ha limiti di frequenza ma non quella quota.

**Da verificare all'applicazione:**
- che la lettura tramite vista non abbia quota (lo dice la descrizione dello strumento, non l'ho osservato) e rispetti l'ordinamento per `Creato`;
- l'API Notion per l'hook: filtro per `created_time` e `last_edited_time`, versione dell'API per i data source;
- il nome dello strumento Notion di creazione pagine nella configurazione del PC (serve al matcher del `PostToolUse`), e se il risultato contiene l'id della pagina creata;
- se i messaggi tra sessioni via Remote Control arrivano subito o restano in attesa di approvazione.

**Template di pagina:** facoltativo. Utile per le righe che Matteo scrive a mano.

**🩺 Integrity Check (mensile)**, controlli in più:
- ogni riga creata dopo l'adozione ha `Riferimento` non vuoto;
- righe "non committato" senza una riga successiva che le risolva con uno sha → **debiti aperti**;
- Attività *In corso* con `Dove` compilato senza righe di Changelog con la stessa `Sessione` da più di 3 giorni → **prese in carico appese**. Il confronto si fa sulla `Sessione` perché Attività e Changelog non sono collegati.

## Costo
- **Scrittura:** una riga di Changelog per modifica reale (una per serie). Due scritture sull'Attività per presa in carico (prendere e rilasciare), più la rilettura.
- **Lettura, con l'hook:** zero token quando non c'è niente di nuovo; circa 25 token per riga riportata. Le viste si leggono prima delle modifiche reali, circa 1-2k token.
- **Lettura, senza l'hook:** una lettura delle due viste a inizio compito e prima delle modifiche reali, circa 1-2k token; 3-4k alla prima lettura e dopo una compaction.
- **Protocollo:** il record *Registro condiviso* si legge una volta per sessione, solo se la sessione fa modifiche reali o prese in carico. Il Core Protocol sempre caricato cresce di circa quattro righe.
- **Hook:** una chiamata a Notion a ogni messaggio (qualche centinaio di millisecondi); uno script da mantenere; un token in sola lettura sul PC.

## Limiti dichiarati
- **Nessun vincolo tecnico:** tutto si regge sul fatto che le sessioni seguano le regole. L'hook porta le informazioni nel contesto ma non obbliga a usarle. Le difese restano la 2(b) nel momento della scrittura e il controllo di integrità a posteriori.
- **Sessioni cloud:** l'hook è configurato sul PC. Nel cloud servirebbe il token come segreto dell'ambiente; fino ad allora valgono le sole regole.
- **Sessione al lavoro da ore senza campanello:** se ne accorge solo alla prossima modifica reale.
- **Due scritture nello stesso istante** (anche due prese in carico) restano possibili, con una finestra di secondi, accettata. Il read-back ne riduce gli effetti.
- **Il registro cresce di più**; le serie restano contenute grazie alla presa in carico.

## Cosa resta delle Proposte A e B
Strumenti facoltativi, solo se servono a un progetto specifico: uno script per la parte git della lettura, il timbro di build per la card, l'anteprima per ogni worker. Nessuno è necessario al sistema.

## Revisioni
**Revisione 2** (corregge la prima stesura):
1. La query SQL prima di ogni modifica consumava una quota limitata, già esaurita → lettura tramite vista.
2. La lettura avveniva solo al momento di scrivere → anche a inizio compito.
3. Chi scrive fuori dal registro era invisibile → 2(b), rilettura dello stato vero.
4. Gli spazi condivisi erano ignorati.
5. Le condizioni non erano osservabili dalla sessione → condizioni concrete.
6. Le modifiche di piattaforma e quelle su più sotto-progetti si perdevano.
7. Una riga per commit duplicava `git log` → una per push o rilascio.
8. Nessuna regola per Notion irraggiungibile.
9. Il confronto tra date come stringhe aveva prodotto un errore reale di analisi.

**Revisione 3** (dopo una revisione indipendente e gli scenari di prova):
1. L'orario *"as of"* del fetch non è il boot della sessione → due pagine se non si ricorda nessuna riga vista.
2. "Ogni scrittura nel checkout principale è reale" era troppo costoso → segnalazione del lavoro in corso.
3. Le serie di rilasci producevano troppe righe.
4. "Compaction → handoff" costringeva a ripartire spesso → handoff solo se i cambiamenti sono troppo estesi.
5. Saltare la (a) "nello stesso turno" lasciava cieca una sessione autonoma per ore.
6. Una riga altrui "non committato" ora ferma i rilasci subito.
7. Il testo nel Core Protocol era troppo lungo → quattro righe più un record di Protocollo.
8. Le affermazioni non verificate sono passate nella lista "da verificare".

**Revisione 4** (idee di Matteo: presa in carico delle Attività e avviso alle sessioni già avviate):
1. Il "lavoro in corso" della rev. 3 stava nel Changelog, violando la matrice di ownership (lo stato di esecuzione appartiene alle Attività) → **presa in carico** con `Sessione` e `Dove`, e l'opzione `Tipo` = *In corso* del Changelog è tolta.
2. `Dove` indica lo spazio **senza il branch**: l'incidente era proprio due sessioni nello stesso checkout su branch diversi.
3. La sessione vecchia che continua dopo un handoff scritto da un'altra ora è coperta: l'Attività non porta più la sua `Sessione`.
4. Una sessione già avviata non può accorgersi da sola dei cambiamenti → **hook di avviso** sul registro e sulle Attività (muto se non c'è niente di nuovo) e **campanello** tra sessioni per quelle al lavoro.
5. Corsa alla presa in carico: read-back dopo la scrittura. Le Attività già *In corso* senza `Sessione` non sono conflitti.
