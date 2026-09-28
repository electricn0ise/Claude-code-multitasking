# Piano di implementazione: Proposta C (registro condiviso)

> **Stato:** da approvare (28/09/2026). Riferimento: [Proposta C, revisione 5](../proposte/proposta-C-registro-condiviso.md).
> Il piano è scritto anche come **handoff**: una sessione nuova lo può eseguire senza altro contesto. È coerente con la regola 3 della proposta stessa, visto che la sessione che l'ha scritto ha un contesto molto lungo.

## Obiettivo
Adottare il **nucleo** della Proposta C nel Sistema Memoria su Notion (fase 1), osservarlo per circa una settimana e poi decidere l'hook (fase 2). **Fatto quando:**
- schema, viste e protocolli sono aggiornati e riletti;
- le sessioni aperte sono state chiuse o istruite;
- la prima riga di Changelog del nuovo sistema esiste.

## Regole per chi esegue
1. **Ogni scrittura su Notion va riletta e confrontata** con l'atteso (regola SCRITTURA del Core Protocol). Un fallimento resta tale finché non è corretto e verificato.
2. **Solo aggiunte** allo schema: nessun rename e nessuna modifica di opzioni select. Il design del sistema registra che un rename di opzioni select azzera i valori.
3. **I synced block si modificano sulla pagina sorgente**, mai tramite un riferimento, e con match a riga singola. Il fetch della sorgente mostra `<synced_block>`; quello di una pagina che lo riferisce mostra `<synced_block_reference>`: lì **non si modifica**, perché il Core Protocol avverte che può troncare il blocco.
   - **Core Protocol:** la sorgente è il record 📐 Protocolli **"Core — AI Project Memory"** (`3cfd8c414014813ca6cdd8d3da02489d`), verificato il 28/09. Il System Control Plane ne mostra solo il riferimento.
   - **Protocollo HA:** probabilmente la sorgente è il record 📐 Protocolli "Home Assistant — Runtime" (`3cfd8c41401481b0941dfb4b6cc77e97`). Va verificato al passo 0 prima di modificare.
4. **Prima di modificare un testo se ne salva la versione attuale** nel repo (`backup/`), per poterla ripristinare.
5. **Sorpresa rispetto a questo piano → STOP** e chiedi a Matteo.
6. Questo lavoro è a sua volta una modifica reale al Sistema Memoria. Si chiude con una riga di Changelog, così il sistema si usa fin dal primo giorno.

## Fase 1: il nucleo

### Passo 0: prerequisiti (sola lettura)
- Leggere la Proposta C, revisione 5: sezioni "Definizioni", "Le tre regole", "Modifiche al Core Protocol", "Modifiche allo schema".
- Verificare che nel Changelog i campi `Riferimento` e `Creato` non esistano ancora, e nelle Attività `Sessione` e `Dove`. (Verificato il 27/09: non esistono, e i nomi non collidono.)
- Individuare la **pagina sorgente** di ogni blocco da modificare (regola 3) e salvare in `backup/<data>/` il testo attuale di:
  - il synced block del Core Protocol, dalla sorgente "Core — AI Project Memory";
  - il protocollo "Home Assistant — Runtime", dalla sua sorgente (il fetch deve mostrare `<synced_block>`);
  - le istruzioni del controllo di integrità mensile. Chiedere a Matteo **dove sono configurate**: il System CP dice "contesto completo sulla pagina del DB", ma la pagina non mostra istruzioni, quindi probabilmente stanno nell'attività ricorrente di ChatGPT.

### Passo 1: schema 🕘 Changelog (`collection://f1c27200-7819-415d-a64c-da6ade9c4ebc`)
- Aggiungere `Riferimento`, di tipo testo.
- Aggiungere `Creato`, di tipo *created time*.
- **Verifica:** fetch del data source; poi lettura di una riga esistente, che deve avere `Creato` compilato.

### Passo 2: schema 📋 Attività (`collection://a49077bf-c915-4f51-b7f7-5346e3330383`)
- Aggiungere `Sessione` e `Dove`, tutti e due di tipo testo.
- **Verifica:** fetch del data source.
- **Presa in carico dell'adozione.** In questo progetto Notion è il prodotto, quindi i passi 3-7 sono modifiche reali. Si usa il sistema su se stesso:
  - creare l'Attività "Adozione registro condiviso", sotto il progetto 🧠 Sistema Memoria — Redesign (approvata da Matteo insieme a questo piano);
  - prenderla in carico con `Sessione` = questa sessione e `Dove` = "Sistema Memoria: schema e protocolli";
  - rileggere l'Attività (read-back). Verrà rilasciata al passo 8.

### Passo 3: le due viste
- **Changelog → "Registro recente"**: tabella ordinata per `Creato` dal più recente, senza filtri; colonne `Nome`, `Tipo`, `Riferimento`, `Sotto-progetto`, `Progetto`, `Sessione`, `Creato`.
- **Attività → "Attività in corso"**: tabella con filtro `Stato` = *In corso*; colonne `Nome`, `Sotto-progetto`, `Sessione`, `Dove`.
- **Verifica** (sono i primi punti "da verificare" della proposta):
  - leggere ogni vista in modalità `view` con `page_size` 5: le righe devono arrivare ordinate per `Creato` e contenere `Creato` e `Riferimento`;
  - annotare gli URL delle due viste: vanno scritti nel record di Protocollo.
- **Quota: detto chiaramente.** `get_tool_access` restituisce solo uno stato (*available_with_limit*), non un contatore, quindi qui **non si può dimostrare** che la lettura tramite vista non consumi quota. Lo si scopre usandola, nella settimana di osservazione. Se una lettura fallisce per quota vale come "Notion irraggiungibile": niente modifiche reali senza Matteo. Se succede davvero, la fase 2 (l'hook, che usa l'API REST con il proprio token e non quella quota) diventa **necessaria**, non più facoltativa.

### Passo 4: record 📐 Protocolli "Registro condiviso" (`collection://3720e04d-e16f-4c01-b85f-1fdfe9d28c69`)
- Proprietà: `Nome` = "Registro condiviso", `Ambito` = *Core*, `Attivo` = sì, `Chiave` = "registro-condiviso".
- Corpo: le sezioni "Definizioni" e "Le tre regole" della proposta, più la procedura di lettura con gli **URL delle due viste** (passo 3), `page_size` 5 e il criterio di stop. Più una riga che serve alla settimana di osservazione: *"se la regola 2 ti ferma o ti fa integrare qualcosa, scrivilo nella riga di Changelog o nell'Attività"*.
- **Non** includere la sezione sull'hook: entra solo in fase 2.
- **Verifica:** fetch della pagina, che deve essere di circa 1,5-2,5k token. Se è molto più lunga, va condensata.

### Passo 5: Core Protocol (synced block nella sorgente "Core — AI Project Memory")
Tre sostituzioni a riga singola e due aggiunte, con il testo esatto della proposta, sezione "Modifiche al Core Protocol":
1. **DURANTE**: la riga "Non reinterrogare Notion per ogni file/commit…" diventa il paragrafo breve, con due condizioni:
   - il paragrafo contiene un **link diretto (mention) al record *Registro condiviso***. Il BOOT vieta `notion-search`, quindi senza link una sessione non troverebbe il record né gli URL delle viste;
   - la parte sull'hook si omette in fase 1.
2. **FINE SESSIONE**: nella riga del Changelog si sostituisce **solo la prima frase** ("1 record solo per eventi consequenziali (…). Append-only."). La frase successiva sul campo `Backup` e su `Rollback target` **resta**: il controllo mensile la usa.
3. **MATRICE DI OWNERSHIP**: la riga "Stato di esecuzione…".
4. **AUTONOMO**: prendere e rilasciare in carico è autonomo. Per aprire un'Attività in autonomia, applicare **solo se Matteo lo approva** (è una raccomandazione della proposta).
5. **Regola generale**: una modifica al Core Protocol vale solo per le sessioni avviate dopo.

**Verifica:** fetch del System CP; il testo nuovo c'è, il resto del blocco è intatto (confrontarlo con il backup del passo 0).

### Passo 6: protocollo Home Assistant — Runtime
Aggiungere la riga: *"Sotto-progetto di piattaforma: HA-core. Le sue righe di Changelog contano per tutte le sessioni HA (Protocollo Registro condiviso)."* Verifica con fetch.

### Passo 7: istruzioni del controllo di integrità mensile
Dove Matteo ha indicato (passo 0), aggiungere tre controlli:
- ogni riga creata dopo l'adozione ha `Riferimento` non vuoto;
- le righe "non committato" non saldate da una riga successiva ("salda il non committato del <data>") sono debiti aperti;
- le Attività *In corso* con `Dove` compilato e senza righe di Changelog con la stessa `Sessione` da più di 3 giorni sono prese in carico appese.

### Passo 8: riga di Changelog dell'adozione
- `Nome`: "Sistema Memoria: adottato il registro condiviso (Proposta C, fase 1)"
- `Tipo`: *Milestone*
- `Progetto`: 🧠 Sistema Memoria — Redesign
- `Riferimento`: `Claude-code-multitasking@<sha>`
- `Sessione`: la sessione che esegue il piano

È l'unica riga della serie di modifiche dei passi 3-7, fatte sotto presa in carico (regola 1). Poi si **rilascia** l'Attività "Adozione registro condiviso": `Stato` = Fatto, `Sessione` vuota.

**Verifica:** la riga compare in cima alla vista *Registro recente*, e l'Attività non compare più in *Attività in corso*.

### Passo 9: passo di adozione (Matteo)
- Chiudere tutte le sessioni Claude Code aperte. Quelle da tenere vanno istruite a rileggere il System Control Plane.
- Sessioni note ancora collegate al 27/09: "Dashboard 3d casa" (`session_01EMvfx…`) e "Dashboard 3D casa ciclo solare" (`session_014pxKBy…`).
- Da qui in poi, ogni sessione nuova nasce con le regole.

## Settimana di osservazione (circa 7 giorni)
Una sessione, oppure Matteo, raccoglie alla fine:

| Misura | Come |
|---|---|
| Righe di Changelog e quante con `Riferimento` | query o vista |
| Prese in carico: quante, quante appese, conflitti trovati | vista *Attività in corso* e Changelog |
| Casi in cui la 2(a) o la 2(b) ha fermato una scrittura | righe di Changelog e Attività (il record di Protocollo chiede di annotarli) |
| Letture tramite vista fallite per quota | note delle sessioni |
| Regole saltate | controllo di integrità anticipato a fine settimana |
| Costo reale in token di una lettura della 2(a) | una misura su una sessione |

**Decisione a fine settimana:** tenere il nucleo così com'è, correggerlo, o procedere con la fase 2. Va scritta come Decisione su Notion. **Criteri per la fase 2** (ne basta uno):
- una sessione ripresa ha mancato un cambiamento che l'hook le avrebbe segnalato;
- le letture della 2(a) pesano più del 10% dei token di una sessione tipica;
- una lettura tramite vista è fallita per quota.

## Fase 2: l'hook (solo dopo la decisione)
Piano sintetico, da dettagliare allora:
1. **Integrazione Notion in sola lettura**, collegata a Changelog, Attività, Sotto-progetti e Progetti. Token in una variabile d'ambiente del PC. Verificare che non scada.
2. **Script `hook-registro.py`**: solo libreria standard, API REST di Notion (data source).
   - Controlli su `UserPromptSubmit` e su `PostToolUse` (al massimo ogni 10 minuti).
   - Finestra sovrapposta di 5 minuti, doppioni per id.
   - Esclusione delle pagine scritte dalla sessione, tramite `PostToolUse` sugli strumenti Notion.
   - Riga di conferma con autocontrollo, riga di errore invece del silenzio.
   - Una riga raggruppata per sotto-progetto, con l'intero progetto a parte.
   - Pulizia dei file di stato più vecchi di 14 giorni.
3. **Test automatici** con l'API di Notion simulata:
   - riga creata vicino a un controllo, anche con ore arrotondate al minuto → riportata;
   - pagina propria → esclusa;
   - pagina propria modificata dopo da altri → riportata;
   - filtro rotto → errore;
   - Notion che non risponde → errore entro 3 secondi;
   - nessuna novità → nessun output.
4. **Verifiche sul PC:**
   - nomi degli strumenti Notion e forma di `tool_response`;
   - che il testo del `PostToolUse` entri nel contesto;
   - arrotondamento dei timestamp;
   - latenza dell'avvio di Python;
   - se un hook nuovo vale per le sessioni già aperte.
5. **Installazione** in `~/.claude/settings.json`, poi di nuovo il passo di adozione.
6. **Core Protocol:** aggiunta della frase sul salto della 2(a) con l'hook confermato attivo, e della sezione sull'hook nel record *Registro condiviso*.

## Rollback
- **Testi** (Core Protocol, protocollo HA, istruzioni del controllo): si ripristinano da `backup/<data>/`.
- **Schema:** i campi aggiunti si eliminano. Nessun dato esistente viene toccato, perché sono solo aggiunte.
- **Viste e record di Protocollo:** si eliminano.
- La riga di Changelog dell'adozione resta: il Changelog è append-only. Si aggiunge una riga di rollback.

## Chi esegue
- **Passi 0-8:** una **sessione Claude Code nuova** con accesso in scrittura a Notion, che parte da questo piano. Può essere locale oppure cloud, perché nella fase 1 non serve il PC. Matteo approva l'inizio e i punti marcati.
- **Passo 9 e settimana di osservazione:** Matteo.
- **Fase 2:** una sessione locale sul PC, perché l'hook si installa lì.
