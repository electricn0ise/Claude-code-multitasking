# PROPOSTA C: il Changelog come registro condiviso (raffinamento del Sistema Memoria)

> **Stato: PROPOSTA preferita, non ancora applicata** (27/09/2026, revisione 3). Nasce dall'appunto di Matteo sulle Proposte [A](proposta-A-controlli-per-sessione.md) e [B](proposta-B-coordinatore-unico-scrittore.md): serve un sistema semplice e indipendente dal tipo di progetto, senza bloccare strumenti specifici.
>
> Non aggiunge database, script o hook. Cambia **quando** si scrive il 🕘 Changelog, aggiunge due letture prima di scrivere, due campi e un'opzione al Changelog, una vista e un record di Protocollo. Le correzioni rispetto alle stesure precedenti sono in fondo, sezione "Revisioni".

## Principio
Il mezzo di coordinamento tra le sessioni è **un unico registro condiviso: il Changelog**. Le sessioni non devono conoscersi né sapere quale sia la "principale".
- Il **registro** dice chi ha cambiato cosa, e chi sta lavorando dove.
- Lo **stato vero** (git, HA, file) dice cosa c'è adesso.

Prima di scrivere si guardano tutti e due.

### Tre definizioni
**Modifica reale:** una scrittura su qualcosa che altre sessioni, o Matteo, usano o leggono:
- push o merge su un branch condiviso;
- rilascio o deploy;
- scrittura in HA (automazioni, dashboard, risorse, config, `/config/www`);
- file definitivi (per esempio `CASA.sh3d`);
- cambio di branch in un checkout usato anche da altri.

Non sono modifiche reali:
- modificare file o committare sul proprio branch;
- il lavoro in un worktree o in una copia propria;
- le anteprime o le entità di prova proprie;
- le scritture su Notion.

**Lavoro in corso:** una riga di registro con `Tipo` = *In corso* che dice "sto lavorando qui" (per esempio "lavoro nel checkout principale di `sweet-home-3d-casa`, branch `sole`", oppure "serie di rilasci della card in corso"). Si chiude con la riga normale di fine lavoro, che la cita. Serve in due casi:
- quando si lavora a lungo in uno **spazio condiviso** (il checkout principale di un repo, una risorsa in copia unica);
- quando si fa una **serie** di modifiche reali alla stessa cosa (per esempio rilasci ripetuti per provare sul tablet).

Nel secondo caso le modifiche intermedie non hanno una riga ciascuna: basta la riga *In corso* all'inizio e quella finale con il riferimento. L'alternativa al lavoro in corso nello spazio condiviso è un worktree proprio.

**Base non affidabile:** la sessione non può più fidarsi di ciò che ha in contesto. Succede in tre casi:
- Matteo lo dice;
- dopo una compaction, la regola 2 mostra cambiamenti altrui troppo estesi per integrarli rileggendo pochi file;
- la sessione è stata avviata prima dell'adozione di questo sistema.

## Le tre regole

**1. Scrivi nel registro subito dopo ogni modifica reale** (o all'inizio e alla fine di un lavoro in corso). Una riga con:
- `Nome`: cosa e la versione, se c'è;
- `Riferimento`: sha, `backup_id` o percorso, altrimenti "non committato";
- `Sotto-progetto`: tutti quelli toccati, oppure vuoto con solo `Progetto` se la modifica è di piattaforma;
- `Tipo` e `Sessione`, come oggi.

Se non riesci a scriverla (per esempio Notion è irraggiungibile), non fai altre modifiche reali finché non ci riesci, a meno che Matteo non decida diversamente. Tieni l'elenco di ciò che manca e lo registri appena puoi.

**2. All'inizio di ogni compito e prima di una modifica reale, leggi due cose.**
- **(a) Il registro.** Si legge la vista *Registro recente* (vedi sotto), dall'alto, fino alla **prima riga che hai già visto** in una lettura precedente. Se non ricordi nessuna riga vista (sessione nuova, o dopo una compaction), leggi due pagine. Delle righe lette contano quelle del tuo sotto-progetto, o di piattaforma del tuo progetto, non scritte da te. In particolare:
  - un **lavoro in corso** di un'altra sessione sullo stesso spazio o sulla stessa cosa → usi un worktree proprio, oppure chiedi;
  - una riga altrui con "non committato" → la produzione è avanti rispetto a git: prima di qualunque rilascio va integrata.
- **(b) Lo stato vero di ciò che stai per sovrascrivere.** Rileggi il file live, la config, la punta del branch remoto (`git fetch`) e, in uno spazio condiviso, controlla se ci sono modifiche non tue (`git status`). Serve contro chi scrive fuori dal registro: Matteo dall'interfaccia, aggiornamenti automatici, righe dimenticate.

Se (a) o (b) mostrano cambiamenti non tuoi, **prima integri e poi scrivi** (merge, rilettura, adattamento), oppure chiedi a Matteo.

Eccezioni per non ripetere le letture inutilmente:
- dentro lo stesso turno di lavoro (senza nuovi messaggi di Matteo) e per **modifiche reali consecutive alla stessa cosa**, la (a) si può saltare; la (b) resta sempre;
- dopo una compaction si rifanno (a) e (b) per intero.

**3. Con una base non affidabile non si fanno modifiche reali: si scrive un handoff e si riparte.** L'handoff va nell'Attività:
- `Handoff` = → Claude Code;
- `Contesto handoff`: 1-2 righe;
- il documento completo nel corpo sotto `## Handoff <data>`: obiettivo misurabile, fatti con evidenze, regole, fasi con condizioni di STOP, fuori scope, "fatto quando".

La sessione che ha scritto un handoff non fa più modifiche reali. Chiude anche i propri lavori in corso, oppure li lascia scritti esplicitamente nell'handoff.

## Scenari di prova (percorsi sulla carta)
| Scenario | Cosa succede | Esito |
|---|---|---|
| **Due sessioni vive nello stesso checkout** (l'incidente del 26-27/09) | A registra "lavoro in corso nel checkout principale, branch `sole`". B, a inizio compito, lo vede con la 2(a): lavora in un worktree proprio oppure chiede. Se A ha dimenticato la riga, B la 2(b) la fa comunque prima di scrivere: `git status` mostra modifiche non sue → si ferma. | regge |
| **La sessione ripresa non sa del ciclo solare** (il sintomo segnalato) | A inizio compito la 2(a) mostra le righe v64-v68 di un'altra sessione: la sessione si aggiorna prima di ragionare, non al momento di scrivere. | regge |
| **Rilascio da codice non committato** (v71) | La riga porta "non committato". Chiunque legge sa che la produzione è avanti rispetto a git e integra prima di un rilascio; il controllo di integrità lo elenca come debito. | regge |
| **Matteo modifica un'automazione dall'interfaccia**, poi una sessione fa remove + recreate | Nel registro non c'è niente, ma la 2(b) rilegge la config live subito prima di scrivere, vede che è diversa da quella letta prima e integra o chiede. | regge |
| **Sessione ripresa dopo tre giorni** | Senza compaction ricorda l'ultima riga vista e legge fino a lì, quindi vede tutto. Con compaction legge due pagine e la 2(b); se i cambiamenti sono troppo estesi scatta la regola 3. | regge; oltre 30 righe in tre giorni la 2(a) può perderne qualcuna, ma la 2(b) resta la difesa prima di scrivere |
| **Ciclo lungo di prove sul tablet** (render → rilascio → guarda, dieci volte) | Una riga "serie di rilasci in corso", la (a) una sola volta, la (b) prima di ogni rilascio (il file live è ancora il mio ultimo?), una riga finale con il riferimento. Un'altra sessione vede la serie in corso. | regge, con due righe invece di dieci |
| **Notion irraggiungibile a metà di una serie** | Stop alle modifiche reali, oppure Matteo decide di continuare. Elenco delle righe mancanti da registrare al rientro. | regge |
| **Sessione vecchia che continua dopo un handoff scritto da un'altra** (il caso reale del 27/09) | Non la ferma nessuna regola: non sa dell'handoff. La sua riga successiva (commit, rilascio) però è visibile, e la 2(b) della sessione nuova protegge il risultato, come è successo davvero con il controllo md5. | limite dichiarato |

## Modifiche al ⚙️ Core Protocol (sempre caricato: aggiunta corta)
Il Core Protocol è caricato in ogni sessione, e il Sistema Memoria è nato proprio per ridurre quel costo. Per questo nel Core Protocol vanno solo quattro righe; definizioni e dettagli vanno in un record di 📐 Protocolli letto **solo quando serve**.

**DURANTE**. Sostituire:
> Non reinterrogare Notion per ogni file/commit. Tocca Notion a fine sessione, o per una Decisione stabile (append-only).

con:
> Non reinterrogare Notion per ogni file/commit, **salvo il registro condiviso** (🕘 Changelog; regole complete nel Protocollo *Registro condiviso*, da leggere prima della prima modifica reale della sessione): (1) subito dopo ogni modifica reale, una riga con `Riferimento`; (2) a inizio compito e prima di una modifica reale, leggi la vista *Registro recente* e rileggi dallo stato vero ciò che sovrascrivi; (3) base non affidabile → handoff nell'Attività e sessione nuova. Per il resto tocca Notion a fine sessione, o per una Decisione stabile (append-only).

**FINE SESSIONE**. Nella riga del Changelog, sostituire "1 record solo per eventi consequenziali (…)" con:
> le modifiche reali sono già registrate (DURANTE); qui solo gli eventi consequenziali che non lo sono (transizione di stato, milestone, decisione) e la chiusura dei propri lavori *In corso*. Ogni riga ha `Riferimento`.

**Nuovo record in 📐 Protocolli: "Registro condiviso".** Contiene le sezioni "Tre definizioni" e "Le tre regole" di questo documento, più la procedura di lettura della vista. È collegato al Progetto come gli altri protocolli.

## Modifiche al 🕘 Changelog (schema: da approvare)
| Serve | Campo | Stato |
|---|---|---|
| Cosa, progetto, sotto-progetto (anche più di uno), sessione, backup HA | `Nome`, `Progetto`, `Sotto-progetto`, `Sessione`, `Backup`, `Rollback target` | invariati |
| Riferimento (sha, `backup_id`, percorso o "non committato") | **`Riferimento`** (testo) | nuovo |
| Ora esatta di creazione, per ordinare | **`Creato`** (*created time*, automatico, compilato anche per le righe esistenti) | nuovo |
| Lavoro in corso | **`Tipo`: nuova opzione *In corso*** | nuova opzione. Si aggiunge senza rinominare quelle esistenti: il rename azzera i valori, come registrato nel design del sistema |

- `Riferimento` è un campo e non una riga nel corpo perché una lettura via vista o query restituisce le proprietà, **non il corpo delle pagine**.
- `Backup` resta com'è; per una scrittura in HA con backup si compilano tutti e due.
- `Data` resta com'è: ha solo il giorno.

**Vista *Registro recente*:**
- ordinata per `Creato`, dal più recente, senza filtri;
- mostra solo `Nome`, `Tipo`, `Riferimento`, `Sotto-progetto`, `Progetto`, `Sessione`, `Creato`, per tenere bassi i token;
- serve anche a Matteo: un *In corso* rimasto aperto a lungo salta all'occhio.

**Perché leggere tramite la vista e non con una query SQL.** Il workspace non è su piano Business: `query_data_sources` risulta *available_with_limit*, e la quota delle query si è già esaurita una volta (🧠 Sistema Memoria — Design & Storia). Lettura: `query_data_sources` in modalità `view`, `page_size` 15.

**Da verificare all'applicazione** (non ancora osservato):
- che la modalità vista non abbia quota: è quanto dice la descrizione dello strumento, ma non l'ho provato su molte letture;
- che restituisca la proprietà `Creato` e rispetti l'ordinamento della vista. Il 27/09 ho letto solo la vista predefinita, che non ha ordinamento: restituiva le proprietà visibili e `Sotto-progetto`, ma l'ordine osservato non è una garanzia.

**Template di pagina:** facoltativo. Utile per le righe che Matteo scrive a mano, per esempio dopo una modifica dall'interfaccia di HA.

**🩺 Integrity Check (mensile)**, tre controlli in più:
- ogni riga creata dopo l'adozione ha `Riferimento` non vuoto;
- righe "non committato" senza una riga successiva che le risolva con uno sha → **debiti aperti**;
- righe *In corso* senza una riga di chiusura → **lavori appesi**.

## Costo
- **Scrittura:** una riga per modifica reale; per una serie o per un lavoro in uno spazio condiviso, due righe (inizio e fine).
- **Lettura, regime normale:** una lettura della vista a inizio compito e prima delle modifiche reali, circa 1-2k token. La prima lettura di una sessione e quella dopo una compaction sono due pagine, circa 3-4k token.
- **Protocollo:** il record *Registro condiviso* si legge una volta per sessione, solo se la sessione fa modifiche reali. Il Core Protocol sempre caricato cresce di circa quattro righe.
- **Rilettura:** mirata, di ciò che si sovrascrive.
- Niente da configurare per progetto.

## Limiti dichiarati
- **Nessun vincolo tecnico:** il sistema si regge sul fatto che le sessioni seguano le regole. Il caso reale del 27/09 mostra che una sessione può ignorarle. Le difese sono la 2(b), che protegge il momento della scrittura, e il controllo di integrità, che rende visibili righe mancanti, debiti e lavori appesi.
- **Una sessione che ignora un handoff scritto da altri** non viene fermata; il suo lavoro resta però visibile nel registro.
- **Il registro cresce di più.** Le serie di rilasci restano contenute grazie al lavoro in corso.
- **Due scritture nello stesso istante** restano possibili (finestra di secondi, accettata).
- **Pausa lunga con compaction e più di 30 righe nel frattempo:** la lettura del registro può perderne qualcuna; la 2(b) resta la difesa prima di scrivere.

## Cosa resta delle Proposte A e B
Strumenti facoltativi, solo se servono a un progetto specifico: uno script per la parte git della lettura, il timbro di build per la card, l'anteprima per ogni worker. Nessuno è necessario al sistema.

## Revisioni
**Revisione 2** (corregge la prima stesura):
1. La query SQL prima di ogni modifica consuma una quota limitata, già esaurita in passato → lettura tramite vista.
2. La lettura avveniva solo al momento di scrivere, quindi la sessione ripresa poteva lavorare per ore su una base vecchia → lettura anche all'inizio di ogni compito.
3. Chi scrive fuori dal registro era invisibile → 2(b), rilettura dello stato vero.
4. Gli spazi condivisi (checkout comune) erano ignorati.
5. "Ultima lettura" e "ferma o gonfia" non erano osservabili dalla sessione → condizioni concrete.
6. Le modifiche di piattaforma e quelle su più sotto-progetti si perdevano → `Sotto-progetto` multiplo e righe di piattaforma incluse nella lettura.
7. Una riga per ogni commit duplicava `git log` → una riga per push o rilascio.
8. Nessuna regola per Notion irraggiungibile.
9. Il confronto tra date come stringhe aveva prodotto un errore reale di analisi (v64-v67 date per mancanti).

**Revisione 3** (dopo una revisione indipendente e gli scenari di prova):
1. Punto di partenza della prima lettura: l'orario *"as of"* del fetch non è il boot della sessione (il System CP letto il 27/09 riportava 24/09) → due pagine quando non si ricorda nessuna riga vista.
2. "Il checkout principale è sempre condiviso, ogni scrittura è reale" avrebbe reso ogni modifica di file una riga di registro, troppo costoso → **lavoro in corso**: due righe per tratto di lavoro, oppure un worktree.
3. Le serie di rilasci per provare sul tablet avrebbero prodotto dieci righe → una serie in corso, con due righe.
4. "Compaction → handoff" avrebbe costretto a ripartire spesso (la compaction è normale nelle sessioni lunghe) → dopo una compaction si rileggono (a) e (b), e l'handoff solo se i cambiamenti sono troppo estesi.
5. Saltare la (a) "nello stesso turno" lasciava cieca una sessione autonoma per ore → salto consentito solo per modifiche consecutive alla stessa cosa.
6. Una riga altrui "non committato" ora ferma i rilasci subito, senza aspettare il controllo mensile.
7. Il testo nel Core Protocol, sempre caricato, era troppo lungo → quattro righe, e i dettagli nel record di Protocollo *Registro condiviso*.
8. Affermazioni non verificate (quota della vista, ordinamento, `Creato` nella vista) ora sono nella lista "da verificare".
