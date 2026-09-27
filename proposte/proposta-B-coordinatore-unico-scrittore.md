# PROPOSTA B: coordinatore come unico scrittore

> **Stato: PROPOSTA, non approvata e non implementata** (27/09/2026). Nata da un'idea di Matteo e scritta per confrontarla con la [Proposta A](proposta-A-controlli-per-sessione.md). Appunto di Matteo, da tenere presente nel giudizio: rischia di diventare troppo legata a strumenti specifici ("bloccare questo o quel tool") e di rompersi al primo caso fuori dai canoni.

## Idea
Nei progetti grandi lavorano molte sessioni in parallelo, senza una sessione "principale" stabile. In questa proposta si distinguono due ruoli:
- **Worker**: qualunque sessione. Esplora, progetta, sviluppa e prova liberamente, anche sul dispositivo vero.
- **Coordinatore**: una sessione leggera ed **unica autorizzata a fare modifiche reali** al progetto: commit su `main` e branch condivisi, push, scritture in produzione su HA, scrittura dei file definitivi, registrazione nel Changelog.

Vale per ogni tipo di progetto, con o senza repo: "modifica reale" è qualunque scrittura sullo stato condiviso.

## Cosa risolve dell'incidente della Dashboard 3D (27/09)
- v68 e v71 rilasciate in produzione da **due sessioni diverse** → con un solo scrittore non succede.
- Rilasci registrati con disciplina diversa a seconda della sessione (v64-v68 registrati, v71 senza riferimento a un commit, v69 e v70 senza traccia) → l'unico scrittore registra ogni scrittura nello stesso modo. *(Correzione: una prima analisi aveva dato per mancanti anche v64-v67, per un errore nel filtro sulla data.)*
- "Remove + recreate" di un'automazione da una copia vecchia → il confronto con lo stato live prima di scrivere sta **in un posto solo**, invece di essere un'istruzione che ogni sessione può saltare.
- Conferme di Matteo prima di ogni scrittura in HA → concentrate in una sessione invece che sparse.

## Anteprima e produzione
Matteo ha bisogno di vedere spesso le modifiche sul dispositivo vero, per qualunque progetto. Per questo il confine non è "prove o produzione" ma **anteprima reale o produzione**:
- **Anteprima (libera per i worker):** ogni worker ha un suo spazio sul dispositivo vero. Esempi:
  - card: risorsa `/local/<progetto>/anteprima/<worker>/…` e vista "Anteprima" sul tablet;
  - dashboard HA: una copia di anteprima;
  - automazioni: una versione di prova disattivata o con prefisso.
  Spazi separati, quindi due worker non si sovrascrivono.
- **Produzione (solo coordinatore):** il coordinatore **promuove esattamente ciò che Matteo ha visto in anteprima**: gli stessi byte o la stessa config, non una ricostruzione.

## Di che contesto ha bisogno il coordinatore
Non del contesto di **sviluppo** (come è fatta la card, perché si è scelta una soluzione), ma di quello di **decisione**:

| Domanda | Da dove la risposta |
|---|---|
| La modifica è approvata? | Conferma di Matteo sull'anteprima (il protocollo la richiede già). Il coordinatore non giudica il design. |
| È sicura da applicare? | Controlli meccanici: produzione ancora uguale alla base da cui è partito il worker, test OK, backup fatto, branch pulito. |
| Si scontra con altro lavoro? | Richieste aperte, Changelog recente, Decisioni vigenti del sotto-progetto (Notion). |

Quando serve un vero giudizio (due richieste sulla stessa cosa, o una richiesta che contraddice una Decisione vigente) **si ferma e chiede a Matteo**. Lo stato è tutto su Notion, git e HA, quindi un coordinatore diventato pesante si sostituisce con uno nuovo senza perdere nulla.

## Richiesta di modifica (worker → coordinatore)
- Formato: lo stesso handoff della Proposta A. Contiene la modifica, la **versione di partenza** (sha, config letta o md5), dove sta l'anteprima approvata, le verifiche fatte e le condizioni di STOP.
- La richiesta vive in un'**Attività su Notion**: è durevole, visibile a Matteo e sopravvive al cambio di coordinatore. `SendMessage` serve solo a svegliare il coordinatore.
- Consegna **delimitata**: branch o worktree del worker, patch, oppure config proposta. Mai "committa quello che c'è nel checkout", perché si porterebbe dietro il lavoro a metà degli altri, cioè lo stesso meccanismo della v71.

## Il coordinatore, a ogni richiesta
1. Rilegge lo stato del sotto-progetto su Notion: richieste aperte, Changelog recente, Decisioni.
2. Rilegge la produzione live e la confronta con la versione di partenza del worker. Se è diversa: **STOP** e mostra il diff.
3. Chiede o verifica la conferma di Matteo.
4. Promuove l'anteprima approvata: fa backup, applica e verifica.
5. Registra nel Changelog: cosa, dove, sha o backup, sessione. Poi chiude la richiesta.

## Vincoli tecnici (verificati sulla documentazione, non ancora sul PC)
- I tool MCP si possono negare per nome (`mcp__server__tool`, `mcp__server__*`). Un blocco vince su qualunque permesso, quindi va messo **solo** sui worker, non nelle impostazioni utente: con il `.claude/settings.json` della cartella da cui partono, oppure con `--settings` o `--disallowedTools`.
- Lo stesso tool scrive sia in anteprima sia in produzione, quindi un blocco per nome è troppo grossolano. Servirebbe un hook `PreToolUse` che guarda la destinazione.
- I blocchi su comandi Bash si aggirano facilmente (`git -C . push`, `bash -c`). Per git servirebbero gli hook di git (`pre-commit` e `pre-push`) con una variabile di ruolo `CLAUDE_ROLE=coordinatore`.
- Non documentato quali impostazioni valgano per le sessioni avviate dall'app desktop o via Remote Control: va provato sul PC.

## Limiti e rischi
- **Legame con gli strumenti** (appunto di Matteo): la parte di blocco richiede di elencare tool, destinazioni e comandi per ogni progetto, e si rompe ai casi nuovi. Senza blocchi la proposta regge lo stesso, ma diventa di nuovo una questione di disciplina.
- **Anteprima da costruire** per ogni progetto: per la card vuol dire rendere parametrico il nome dell'elemento, perché le risorse Lovelace sono globali.
- **Un passaggio in più** per ogni rilascio in produzione.
- **Collo di bottiglia**: se il coordinatore è occupato, la promozione aspetta.
