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

**Presa in carico.** Un'Attività con `Stato` = *In corso*, il campo `Sessione` compilato (l'URL della sessione; "Matteo" se ci lavora lui a mano; se la sessione non conosce il proprio URL, un nome unico come "locale 27/09 21:40", sempre uguale in tutte le sue righe di Attività e Changelog) e, se tocca uno spazio condiviso, il campo `Dove` compilato. `Dove` indica l'**identità dello spazio, senza il branch**: per esempio "checkout principale sweet-home-3d-casa", "HA produzione: card dashboard3d", "CASA.sh3d". Stesso spazio vuol dire collisione, qualunque sia il branch.
- **Serve** solo per lavorare in uno spazio condiviso, o per una serie di modifiche reali alla stessa cosa (per esempio le prove ripetute sul tablet). Il lavoro in uno spazio proprio non la richiede.
- **Si prende:** si scrive `Sessione` (e `Dove`), poi si rilegge l'Attività, come vuole la regola SCRITTURA ("read-back"). Se c'è un altro valore, ci si ritira.
- **Si rilascia** appena finisce il tratto di lavoro nello spazio condiviso (o la serie), **non a fine sessione**: le sessioni via Remote Control restano aperte per giorni, e una presa in carico tenuta da una sessione ferma blocca le altre senza motivo. Rilasciare vuol dire che `Stato` passa ad altro e `Sessione` si svuota. Anche un handoff è un rilascio: `Handoff` = → Claude Code e `Sessione` vuota. La sessione nuova prende in carico scrivendo la propria.
- **Righe esistenti:** le Attività già *In corso* senza `Sessione` contano come non prese in carico.

**Righe dell'intero progetto.** Contano per **tutte** le sessioni del progetto:
- le righe di Changelog e le Attività **senza sotto-progetto**. È il caso normale per i progetti che non hanno sotto-progetti, e non serve creare un sotto-progetto "Core": il Core Protocol vieta già i sotto-progetti condivisi sintetici;
- le righe del **sotto-progetto di piattaforma**, se il progetto ne ha uno. Per Home Assistant è **HA-core** (config core, integrazioni, aggiornamenti): un aggiornamento di HA riguarda anche chi lavora sull'Antifurto. Il protocollo del progetto lo nomina una volta sola.

**Base non affidabile.** La sessione non può più fidarsi di ciò che ha in contesto. Succede in tre casi:
- Matteo lo dice;
- dopo una compaction, la regola 2 mostra cambiamenti altrui troppo estesi per integrarli rileggendo pochi file;
- la sessione è stata avviata prima dell'adozione di questo sistema.

## Le tre regole

**1. Scrivi nel registro subito dopo ogni modifica reale.** Una riga con:
- `Nome`: cosa, e la versione se c'è;
- `Riferimento`: ciò che identifica **la versione** scritta, così che dopo si possa confrontare con lo stato vero. Per il codice lo sha (`repo@sha`). Per una scrittura in HA il `backup_id`. Per un file fuori da git il percorso più l'md5 (il fix della v71 si è basato proprio sull'md5). Se non esiste, "non committato". Una riga che salda un "non committato" precedente lo dice nel `Riferimento`: "sha …, salda il non committato del <data>";
- `Sotto-progetto`: tutti quelli toccati. Vuoto, con solo `Progetto`, se il progetto non ha sotto-progetti o se la modifica riguarda l'intero progetto e non esiste un sotto-progetto di piattaforma apposito (per HA c'è: HA-core);
- `Progetto`, `Tipo`, `Sessione`.

Per una **serie** alla stessa cosa, sotto presa in carico, basta una riga alla fine, con il riferimento finale. Se la riga non si riesce a scrivere (per esempio Notion è irraggiungibile), niente altre modifiche reali finché non ci si riesce, salvo decisione di Matteo; si tiene l'elenco delle righe mancanti e si registrano appena possibile.

**2. All'inizio di ogni compito e prima di una modifica reale, leggi.** Un **compito** è una richiesta nuova di Matteo: un argomento nuovo, o la ripresa di un lavoro. Non lo sono le risposte e i chiarimenti nello stesso filo di lavoro. Senza hook una sessione non sa distinguere "continua" detto dopo dieci secondi da "continua" detto dopo tre ore: in quel caso se ne accorge prima della modifica reale successiva.
- **(a) Registro e Attività.** Si leggono le viste *Registro recente* e *Attività in corso* (vedi sotto). Il *Registro recente* si legge dall'alto fino alla prima riga già vista; se non ricordi nessuna riga vista (sessione nuova o dopo una compaction), due pagine. Contano le righe non tue del tuo sotto-progetto e quelle **dell'intero progetto** (vedi "Righe dell'intero progetto" sotto). In particolare:
  - **presa in carico di un'altra sessione** sulla stessa cosa, o con lo stesso `Dove` → lavori in uno spazio proprio (worktree, copia, anteprima) finché l'altra non rilascia, oppure chiedi a Matteo;
  - **un'Attività che avevi preso in carico non porta più la tua `Sessione`** (vuota o diversa) → non sei più il proprietario: niente modifiche reali, ti fermi;
  - **una riga altrui "non committato"** → la produzione è avanti rispetto a git: la integri prima di qualunque rilascio.
- **(b) Stato vero di ciò che stai per sovrascrivere.** Rileggi il file live, la config, la punta del branch remoto (`git fetch`) e, in uno spazio condiviso, controlla se ci sono modifiche non tue (`git status`). È la difesa contro chi scrive fuori dal registro: Matteo dall'interfaccia, aggiornamenti automatici, righe dimenticate.

Se (a) o (b) mostrano cambiamenti non tuoi, **prima integri e poi scrivi** (merge, rilettura, adattamento), oppure chiedi a Matteo.

Eccezioni, per non ripetere letture inutili:
- se l'hook è attivo (conferma vista a inizio sessione, nessuna riga di errore) e non ha segnalato novità per il tuo sotto-progetto, la (a) **a inizio compito** si può saltare; prima di una modifica reale si fa comunque;
- per modifiche reali consecutive alla stessa cosa, nello stesso turno di lavoro, la (a) si può saltare; la (b) resta sempre;
- dopo una compaction si rifanno (a) e (b) per intero.

**3. Con una base non affidabile non si fanno modifiche reali: si scrive un handoff e si riparte.** L'handoff va nell'Attività:
- `Handoff` = → Claude Code e `Sessione` vuota;
- `Contesto handoff`: 1-2 righe;
- documento completo nel corpo sotto `## Handoff <data>`: obiettivo misurabile, fatti con evidenze, regole, fasi con condizioni di STOP, fuori scope, "fatto quando".

Chi ha scritto un handoff non fa più modifiche reali.

## L'avviso alle sessioni già avviate
Una sessione non può accorgersi da sola che qualcosa è cambiato: può solo guardare, o essere avvisata. A questo serve **`hook-registro`**: un solo script Python, configurato a livello utente sul PC, che interroga il registro **fuori dal modello**, quindi senza consumare token, e scrive nel contesto solo quando c'è qualcosa da dire.

**Quando controlla**
- **A ogni messaggio di Matteo** (`UserPromptSubmit`). È il caso della dashboard: la sessione era ferma e Matteo ci è tornato sopra.
- **Durante il lavoro autonomo** (`PostToolUse`), al massimo una volta ogni 10 minuti. Tra un controllo e l'altro lo script esce subito, senza chiamare Notion. Copre la sessione che lavora da sola per ore mentre un'altra rilascia.

**Cosa controlla.** Via API REST di Notion, in sola lettura:
- le righe del Changelog create dopo l'ultimo controllo di questa sessione;
- le Attività modificate dopo l'ultimo controllo.

**Non perdere righe.** L'ora dell'ultimo controllo è quella del PC, mentre `created_time` e `last_edited_time` vengono dai server di Notion, e l'API potrebbe arrotondarli al minuto. Un confronto diretto perderebbe le righe nate vicino a un controllo, senza dirlo: è lo stesso tipo di errore che ha nascosto v64-v67. Quindi:
- ogni interrogazione parte **5 minuti prima** dell'ultimo controllo;
- i doppioni si scartano con l'elenco degli id già riportati (per le Attività, id più `last_edited_time`), salvato nello stato dell'hook.

**Esclude ciò che ha fatto la sessione stessa.** Lo stesso script, agganciato in modo silenzioso al `PostToolUse` degli strumenti Notion di scrittura (creazione e modifica di pagine), annota l'id delle pagine che la sessione crea o modifica, con l'ora del server presa dalla risposta dello strumento, se c'è. Una pagina modificata da altri *dopo* viene comunque riportata. **Nel dubbio si riporta**: un falso allarme costa circa 30 token, una riga persa costa l'incidente.

**Cosa scrive**
- **Niente**, se non c'è nulla di nuovo.
- **Una riga, raggruppata per sotto-progetto**, se c'è qualcosa. Per esempio: *"[registro] novità da altre sessioni: Dashboard 3D (2 righe di registro, di cui 1 non committata; 1 Attività); Antifurto (1 riga). Se riguarda il tuo lavoro, leggi le viste (regola 2a)."* L'hook non sa su quale sotto-progetto lavori la sessione (le sessioni HA partono tutte dalla stessa cartella); la sessione lo sa, e decide. Così la riga resta corta anche in un giorno di molte modifiche. Le righe e le Attività senza sotto-progetto vanno sotto il nome del progetto, per esempio *"Home Assistant (intero progetto): 1 riga"*. Quelle e le righe del sotto-progetto di piattaforma (HA-core) riguardano tutte le sessioni del progetto.
- **Una riga di conferma alla prima esecuzione della sessione**: *"[registro] avviso attivo, ultima riga: Dashboard 3D v71, 27/09 16:57"*, più le novità se ce ne sono. Senza questa riga, un hook rotto sarebbe indistinguibile da un hook muto perché non c'è niente di nuovo. Se la conferma non compare, l'hook non funziona e valgono le sole regole.
- **Autocontrollo alla prima esecuzione.** L'hook legge anche l'ultima riga del Changelog **senza filtri**. Se quella riga è più recente dell'inizio della finestra ma la query filtrata non l'ha restituita, il filtro è rotto e scrive la riga di errore invece della conferma. Così un hook che gira ma interroga male se ne accorge da solo, senza chiedere alla sessione di giudicare se una data "sembra vecchia".
- **Una riga di errore** se qualcosa va storto (Notion non risponde entro 2-3 secondi, token scaduto, eccezione dello script): *"[registro] controllo non riuscito: leggi le viste prima di modificare (regola 2a)"*. Mai il silenzio al posto di un errore, e mai un blocco del messaggio.

**Primo controllo di una sessione** (nessuno stato salvato): si parte da **48 ore prima**. Le sessioni avviate prima dell'adozione ricadono comunque nella regola 3.

**Token Notion.** Un'integrazione **in sola lettura**, collegata solo a Changelog, Attività e Sotto-progetti (per scrivere i nomi dei sotto-progetti), con il token in una variabile d'ambiente del PC.

**L'hook è facoltativo.** Senza, il sistema funziona con le sole regole, perché la 2(a) fa lo stesso lavoro spendendo token. Se si rompe, la riga di errore o l'assenza della conferma lo rendono evidente, e si torna alle regole senza rompere nient'altro.

## Scenari di prova (percorsi sulla carta)
| Scenario | Cosa succede | Esito |
|---|---|---|
| **Due sessioni vive nello stesso checkout** (l'incidente del 26-27/09) | La sessione del ciclo solare prende in carico "Ciclo solare" con `Dove` = "checkout principale sweet-home-3d-casa". L'altra, qualunque branch usi, lo vede con l'hook o con la 2(a): worktree proprio, oppure chiede. Se la presa in carico manca, prima di scrivere la 2(b) trova con `git status` modifiche non sue e si ferma. | regge |
| **Matteo torna sulla sessione vecchia e scrive "continua"** (il sintomo segnalato) | Prima che la sessione ragioni, l'hook scrive "Dashboard 3D: 5 righe di registro, 1 Attività". La sessione legge le viste e trova v64-v68 e la presa in carico del ciclo solare. Senza hook, se ne accorge prima della prossima modifica reale (2a). | regge (con l'hook subito, senza hook alla scrittura) |
| **Rilascio da codice non committato** (v71) | La riga porta "non committato". Chi legge sa che la produzione è avanti rispetto a git e la integra prima di un rilascio; il controllo di integrità lo elenca come debito. | regge |
| **Sessione vecchia che continua dopo un handoff scritto da un'altra** (il caso reale del 27/09) | L'handoff ha svuotato `Sessione` e la sessione nuova ha scritto la propria. Alla vecchia l'hook segnala "Dashboard 3D: 1 Attività" al primo messaggio (o entro 10 minuti, se sta lavorando da sola). Leggendo la vista scopre che l'Attività non porta più la sua `Sessione`, e la 2(a) prima del commit la ferma. | regge per le sessioni che avevano preso in carico; quelle avviate prima dell'adozione ricadono nella regola 3 |
| **Matteo modifica un'automazione dall'interfaccia**, poi una sessione fa remove + recreate | Nel registro non c'è niente, ma la 2(b) rilegge la config live subito prima di scrivere, vede la differenza e integra o chiede. | regge |
| **Sessione ripresa dopo tre giorni** | L'hook riassume in una riga, per sotto-progetto, tutto ciò che è cambiato dall'ultimo controllo; la sessione legge le viste di ciò che la riguarda. | regge |
| **L'hook si rompe** (token scaduto, Python aggiornato, API cambiata) | Riga di errore a ogni controllo, oppure manca la conferma a inizio sessione: la sessione torna alle regole, e Matteo vede il problema. | regge, con più token finché non viene sistemato |
| **Ciclo lungo di prove sul tablet** | Presa in carico con `Dove` = "HA produzione: card", 2(b) prima di ogni rilascio, una riga di Changelog alla fine. Le altre sessioni vedono la presa in carico. | regge, con una riga invece di dieci |
| **Sessione al lavoro da ore mentre un'altra rilascia** | Il controllo sul `PostToolUse` la avvisa entro 10 minuti, a metà del lavoro. Comunque la 2(a) e la 2(b) scattano prima della sua prossima modifica reale. | regge (ritardo massimo 10 minuti) |
| **Presa in carico appesa** (la sessione è morta, o è ferma e si è dimenticata di rilasciare) | L'Attività resta *In corso* con una `Sessione` inattiva. Chi la trova lavora in uno spazio proprio o chiede a Matteo; il controllo mensile la segnala. Il rilascio a fine tratto di lavoro, non a fine sessione, rende il caso raro. | regge |
| **Filtro dell'hook rotto** (la query non restituisce righe che esistono) | Alla prima esecuzione l'autocontrollo trova l'ultima riga senza filtri, vede che la query filtrata non l'ha restituita e scrive la riga di errore: la sessione torna alle regole. | regge |
| **Notion irraggiungibile** | L'hook scrive la riga di errore; stop alle modifiche reali finché la riga di registro non si può scrivere, salvo decisione di Matteo. | regge |

## Modifiche al ⚙️ Core Protocol (sempre caricato: aggiunta corta)
Nel Core Protocol vanno solo quattro righe. Definizioni e dettagli vanno in un record di 📐 Protocolli letto **solo quando serve**.

**DURANTE**. Sostituire:
> Non reinterrogare Notion per ogni file/commit. Tocca Notion a fine sessione, o per una Decisione stabile (append-only).

con:
> Non reinterrogare Notion per ogni file/commit, **salvo il coordinamento tra sessioni** (regole complete nel Protocollo *Registro condiviso*, da leggere la prima volta che una di queste regole serve nella sessione): (1) subito dopo ogni modifica reale, una riga di Changelog con `Riferimento`; (2) a inizio compito e prima di una modifica reale, leggi le viste *Registro recente* e *Attività in corso* (a inizio compito si salta se l'hook `[registro]` è confermato attivo e non segnala il tuo sotto-progetto) e rileggi dallo stato vero ciò che sovrascrivi; per uno spazio condiviso prendi in carico l'Attività (`Sessione`, `Dove`); (3) base non affidabile → handoff e sessione nuova. Per il resto tocca Notion a fine sessione, o per una Decisione stabile (append-only).

**FINE SESSIONE**. Nella riga del Changelog, sostituire "1 record solo per eventi consequenziali (…)" con:
> le modifiche reali sono già registrate (DURANTE); qui solo gli eventi consequenziali che non lo sono (transizione di stato, milestone, decisione). Rilascia le prese in carico eventualmente rimaste (vanno rilasciate già a fine tratto di lavoro), oppure lasciale esplicitamente in un handoff. Ogni riga ha `Riferimento`.

**MATRICE DI OWNERSHIP**. Sostituire "Stato di esecuzione di una fetta di lavoro → **Attività**" con:
> Stato di esecuzione di una fetta di lavoro → **Attività** (inclusa la presa in carico: `Sessione`, `Dove`)

**AUTONOMO vs DA PROPORRE.** Prendere e rilasciare in carico un'Attività è **autonomo**, solo comunicato. Raccomandazione, da approvare: anche **aprire un'Attività** diventa autonomo quando sta dentro un sotto-progetto esistente e serve a prendere in carico il lavoro che la sessione sta per fare. Altrimenti le sessioni salterebbero la presa in carico per evitare il passaggio di conferma.

**Nuovo record in 📐 Protocolli: "Registro condiviso".** Contiene le sezioni "Definizioni", "Le tre regole" e "L'avviso alle sessioni già avviate" di questo documento.

**Protocollo 📐 Home Assistant — Runtime.** Una riga in più: *"Sotto-progetto di piattaforma: HA-core. Le sue righe di Changelog contano per tutte le sessioni HA (Protocollo Registro condiviso)."*

## Modifiche allo schema (da approvare)
**🕘 Changelog**
| Campo | Stato |
|---|---|
| `Nome`, `Progetto`, `Sotto-progetto` (anche più di uno), `Tipo`, `Sessione`, `Backup`, `Rollback target`, `Data` | invariati |
| **`Riferimento`** (testo): versione scritta (`repo@sha`, `backup_id`, percorso + md5) o "non committato" | nuovo |
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
- l'API Notion per l'hook: filtri per `created_time` e `last_edited_time`, versione dell'API per i data source, limiti di frequenza (ogni controllo fa 2-3 richieste);
- i nomi degli strumenti Notion di creazione e modifica pagine nella configurazione del PC (servono al matcher del `PostToolUse`), e che `tool_input`/`tool_response` contengano l'id della pagina. La documentazione conferma che `tool_response` è disponibile, ma non la sua forma per questi strumenti;
- che il testo restituito da un `PostToolUse` via JSON (`hookSpecificOutput.additionalContext`) entri davvero nel contesto a metà di un compito: è documentato, non provato;
- se l'API di Notion arrotonda `created_time` e `last_edited_time` al minuto: la finestra di 5 minuti copre entrambi i casi, ma va saputo.

**Template di pagina:** facoltativo. Utile per le righe che Matteo scrive a mano.

**🩺 Integrity Check (mensile)**, controlli in più:
- ogni riga creata dopo l'adozione ha `Riferimento` non vuoto;
- righe "non committato" non saldate da una riga successiva (quella che nel `Riferimento` dice "salda il non committato del <data>") → **debiti aperti**;
- Attività *In corso* con `Dove` compilato senza righe di Changelog con la stessa `Sessione` da più di 3 giorni → **prese in carico appese**. Il confronto si fa sulla `Sessione` perché Attività e Changelog non sono collegati.

## Costo
- **Scrittura:** una riga di Changelog per modifica reale (una per serie). Due scritture sull'Attività per presa in carico (prendere e rilasciare), più la rilettura.
- **Lettura, con l'hook:** zero token quando non c'è niente di nuovo; una riga di conferma a inizio sessione; circa 30-50 token per ogni riga di novità. Le viste si leggono solo quando l'hook segnala novità sul proprio sotto-progetto e prima delle modifiche reali, circa 1-2k token.
- **Lettura, senza l'hook:** una lettura delle due viste a ogni compito (richiesta nuova, non ogni messaggio) e prima delle modifiche reali, circa 1-1,5k token; 3-4k alla prima lettura e dopo una compaction. Con molti compiti al giorno è il costo più alto del sistema, ed è il motivo principale della fase 2.
- **Protocollo:** il record *Registro condiviso* si legge una volta per sessione, solo se la sessione fa modifiche reali o prese in carico. Il Core Protocol sempre caricato cresce di circa quattro righe.
- **Hook:** 2-3 richieste a Notion a ogni messaggio e al massimo ogni 10 minuti durante il lavoro autonomo (qualche centinaio di millisecondi ciascuna). Inoltre l'avvio di Python a ogni chiamata di strumento, anche quando non interroga Notion: circa 50-100 ms su Windows, da misurare. Se pesa, il `PostToolUse` si limita agli strumenti che precedono le scritture (Bash, Edit, Write, MCP). Uno script da mantenere, un token in sola lettura sul PC.

## Limiti dichiarati
- **Nessun vincolo tecnico:** tutto si regge sul fatto che le sessioni seguano le regole. L'hook porta le informazioni nel contesto ma non obbliga a usarle. Le difese restano la 2(b) nel momento della scrittura e il controllo di integrità a posteriori.
- **Sessioni cloud:** l'hook è configurato sul PC. Nel cloud servirebbe il token come segreto dell'ambiente; fino ad allora valgono le sole regole.
- **Ritardo durante il lavoro autonomo:** fino a 10 minuti tra un cambiamento altrui e l'avviso; comunque prima della modifica reale successiva.
- **La riga dell'hook è per sotto-progetto, non per argomento:** due sessioni sullo stesso sotto-progetto ma su cose diverse si vedono a vicenda come novità. È un rumore basso e voluto, perché la scelta di cosa è rilevante resta alla sessione.
- **Due scritture nello stesso istante** (anche due prese in carico) restano possibili, con una finestra di secondi, accettata. Il read-back ne riduce gli effetti.
- **Il registro cresce di più**; le serie restano contenute grazie alla presa in carico.

## Adozione in due fasi
1. **Nucleo:** le tre regole, la presa in carico nelle Attività, i campi nuovi, le due viste, le quattro righe nel Core Protocol e il record di Protocollo. Funziona da solo, ed è già un sistema completo.
2. **Dopo circa una settimana di uso reale:** l'hook `hook-registro`. La settimana serve a verificare il nucleo e dà un punto di confronto per capire quanto l'hook fa risparmiare in token e quanto riduce le sorprese.

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
4. Una sessione già avviata non può accorgersi da sola dei cambiamenti → **hook di avviso** sul registro e sulle Attività, muto se non c'è niente di nuovo.
5. Corsa alla presa in carico: read-back dopo la scrittura. Le Attività già *In corso* senza `Sessione` non sono conflitti.

**Ricontrollo totale della revisione 4** (problemi trovati e corretti prima di chiuderla):
1. **Rumore:** filtrare per progetto avrebbe avvisato ogni sessione HA di ogni rilascio di ogni sotto-progetto, perché le sessioni HA partono tutte dalla stessa cartella → **una sola riga raggruppata per sotto-progetto**, e la sessione decide cosa la riguarda.
2. **Silenzio ambiguo:** un hook rotto era indistinguibile da un hook senza novità, e le sessioni avrebbero saltato la lettura fidandosi del silenzio: di nuovo l'incidente → **riga di conferma** a inizio sessione e **riga di errore** al posto del silenzio. Il salto della 2(a) a inizio compito è ammesso solo con l'hook confermato attivo.
3. **Sessioni al lavoro:** il campanello tra sessioni (`SendMessage`) dipendeva dalla disciplina di chi rilascia → sostituito dallo stesso hook sul `PostToolUse`, al massimo ogni 10 minuti. La documentazione conferma che un `PostToolUse` può aggiungere testo al contesto via JSON.
4. **Punto di partenza del primo controllo:** i timestamp interni del transcript non sono documentati e il formato cambia tra versioni → 48 ore; le sessioni avviate prima dell'adozione ricadono nella regola 3.
5. **Righe proprie:** escludere solo le pagine *create* dalla sessione riportava come novità le sue stesse prese in carico → si escludono anche le pagine *modificate* dalla sessione, a meno che qualcun altro le modifichi dopo.
6. **Identità della sessione:** una sessione che non conosce il proprio URL non potrebbe compilare `Sessione` in modo coerente → usa un nome unico, sempre uguale in tutte le sue righe.
7. **Righe perse in silenzio:** confrontare l'ora del PC con i timestamp dei server Notion (forse arrotondati al minuto) perdeva le righe nate vicino a un controllo, e la conferma non se ne sarebbe accorta → finestra sovrapposta di 5 minuti, doppioni scartati per id, "nel dubbio si riporta"; la conferma nomina l'ultima riga vista.
8. **Salto della 2(a) irraggiungibile:** la regola stava solo nel record di Protocollo, che si legge dopo il primo compito → la regola va anche nelle quattro righe del Core Protocol.
9. **Troppo tutto insieme:** adozione in due fasi, con l'hook solo nella seconda (vedi "Adozione").
10. **Progetti senza sotto-progetti e piattaforma** (osservazione di Matteo): il documento non diceva come trattare le righe senza sotto-progetto, e una sessione sull'Antifurto poteva ignorare un aggiornamento di HA-core perché "non è il suo sotto-progetto" → **righe dell'intero progetto**: quelle senza sotto-progetto e quelle del sotto-progetto di piattaforma (HA-core) contano per tutte le sessioni. Nessun sotto-progetto "Core" obbligatorio, perché il Core Protocol vieta i sotto-progetti condivisi sintetici e i progetti leggeri non ne hanno bisogno.

**Seconda revisione completa** (27/09 sera):
1. **Autocontrollo invece di giudizio:** "la conferma nomina una riga palesemente vecchia" era soggettivo (dopo una settimana tranquilla l'ultima riga è vecchia anche con un hook sano) e non rilevava un filtro rotto → alla prima esecuzione l'hook confronta la query filtrata con l'ultima riga senza filtri e scrive un errore se non tornano.
2. **Prese in carico tenute per giorni:** il rilascio "a fine sessione" non funziona con sessioni via Remote Control aperte per giorni → rilascio a fine tratto di lavoro.
3. **Riferimento verificabile:** un percorso da solo non identifica una versione → percorso più md5 per i file fuori da git; e una riga che salda un "non committato" lo dichiara, così il controllo mensile può abbinarle.
4. **"Worktree proprio" non vale per HA** → "spazio proprio (worktree, copia, anteprima)".
5. **Costo nascosto:** leggere le viste a ogni messaggio sarebbe costato circa 1,3k token a messaggio → "compito" definito come richiesta nuova, e costo dichiarato come motivo principale della fase 2.
6. **Latenza non dichiarata:** l'hook sul `PostToolUse` avvia Python a ogni chiamata di strumento → costo dichiarato, e matcher restringibile.
7. **Lettura del record di Protocollo:** era prevista "prima della prima modifica reale", ma la procedura di lettura (fino a quale riga) serve già al primo compito → si legge la prima volta che una regola serve.
