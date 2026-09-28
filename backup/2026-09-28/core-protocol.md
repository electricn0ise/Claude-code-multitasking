# Backup: Core Protocol (prima dell'adozione della Proposta C)

Sorgente: record 📐 Protocolli "Core — AI Project Memory" (`3cfd8c414014813ca6cdd8d3da02489d`), synced block `#0ea5cdaf52914253bbfd9bcfe7e2eb41`. Letto il 28/09/2026 (fetch "as of 2026-09-06T08:29:38Z", ultima modifica della pagina).

---

### BOOT
1. `CLAUDE.md` → questo System Control Plane. Da qui: fatti utente, questo protocollo, indice progetti. Il fetch del System CP è la **primissima** azione della sessione, anche se parte già in Plan Mode o in un altro workflow builtin (un hook `SessionStart` lo ricorda; il workflow non fa da gate).
2. Abbina la working directory al `Repo` di un progetto. Ambiguo o assente → chiedi, non indovinare.
3. Se la sessione entra nel merito di un progetto: fetch della sua pagina Progetto → Project Control Plane completo.
4. **Naviga cataloghi e URL. Niente ****`notion-search`** salvo fallimento di risoluzione identità.
5. Fetch mirato di un record solo quando serve il suo body (Conoscenza / Decisione / Changelog).
### ATTIVAZIONE (lato [claude.ai](http://claude.ai))
- Su [claude.ai](http://claude.ai) (niente `Repo`): attiva il sistema solo su trigger esplicito (l'utente nomina il sistema o un progetto) o sospetto fondato (stato/decisione concreta, non menzione di passaggio). Max 1 verifica per cambio argomento.
- Se un'Attività in attesa è superata dallo Snapshot o il progetto è In pausa: verifica con l'utente. **La conversazione in corso vince sul salvato.**
- [claude.ai](http://claude.ai) propone (nuovi progetti/sotto-progetti/Attività, contesto); Claude Code esegue (codice, file, runtime, Changelog).
### DURANTE
- Non reinterrogare Notion per ogni file/commit. Tocca Notion a fine sessione, o per una Decisione stabile (append-only).
- Ogni fatto mutabile ha **un solo proprietario** (matrice sotto). Non duplicare stato.
- Ambiguità di identità / ownership / valore → **fermati prima di scrivere**.
### SCRITTURA
- Mutazione minima: cambia solo ciò che serve.
- Stato volatile (Snapshot / Next / Versione) = **edit dell'headline block** dell'entità — 1 write propaga a ogni control plane che lo referenzia.
- Un synced block (headline, questo Core Protocol, i Protocolli) si edita **sulla sua pagina sorgente** (il blocco `<synced_block>`), **mai** attraverso un `<synced_block_reference>` su un altro CP: l'edit via riferimento può troncare il blocco. Match a riga singola, un blocco per volta.
- Nuova Conoscenza = record + 1 riga nel Catalogo Conoscenza del Project CP; se `Caldo`, anche un `<synced_block_reference>` nella sezione Fatti caldi. Cestinare un record referenziato → rimuovere anche la sua riga catalogo e il suo `<synced_block_reference>` dai CP (il ref morto non rompe la pagina ma resta come blocco vuoto con avviso).
- Ogni write consequenziale: **read-back + confronto** atteso/effettivo. Un fallimento resta fallimento finché non corretto e verificato.
- Idempotenza: cerca un record equivalente prima di crearne uno nuovo.
- **Fatto condiviso da più sotto-progetti**: un solo proprietario, al livello più basso che lo contiene per intero. Simmetrico tra ≥2 sotto-progetti → Ambito `Progetto`. Posseduto da uno e usato da un altro → resta nel proprietario + 1 riga-puntatore (non una copia) nel catalogo del consumatore. Co-possesso reale → relation `Sotto-progetto` = entrambi, con parsimonia (2 righe da mantenere e da pulire alla cancellazione). Mai duplicare il contenuto nei body; mai un sotto-progetto "condiviso" sintetico. Decisione che tocca due sotto-progetti → vive in quello che decide e cita l'altro.
### FINE SESSIONE
- Aggiorna gli headline block toccati (sotto-progetti + progetto).
- Changelog: 1 record solo per eventi consequenziali (transizione di stato, cambio versione, milestone/release, fix significativo, decisione). Append-only. Se l'evento ha preso un backup (tarball HA), compila il campo `Backup` (nome + id) e spunta `Rollback target` se è uno stato noto-buono da tenere.
- Handoff sulle Attività: aggiorna `Stato`; se risolta svuota `Handoff` + sposta il `Contesto handoff` in nota in-page; se serve una decisione/contesto dell'utente → `Handoff = → claude.ai`. Il contesto handoff è **azionabile** (cosa serve a chi riprende), non un resoconto. Verifica che non esista già un'Attività equivalente in attesa.
### MATRICE DI OWNERSHIP
- Stato / Salute / Protocollo (relation) / Repo → proprietà del **Progetto**
- Snapshot / Next / Versione → **headline block** dell'entità (1 write propaga a ogni CP)
- Stato di esecuzione di una fetta di lavoro → **Attività**
- Conoscenza durevole (fragilità, quirk, convenzioni) → **Conoscenza** (body del record)
- Perché di una scelta → **Decisioni** (append-only)
- Cosa è cambiato → **Changelog** (append-only)
- Backup HA (nome + `backup_id`, contesto, rollback) → campo **Backup** + checkbox **Rollback target** sul record **Changelog** dell'evento · inventario device fisici → **Dispositivi**
- Stato runtime live (on/off, temp, config effettiva) → **Home Assistant**, mai Notion
### AUTONOMO vs DA PROPORRE
- **Autonomo, solo comunicato**: aggiornare headline/Snapshot/Next/Versione di entità esistenti; chiudere un Handoff; aggiungere una riga a Changelog; aggiornare il catalogo.
- **Da proporre prima**: nuovo Progetto o Sotto-progetto; Stato → Concluso/Archiviato; nuova Attività; cambio di ownership o schema.
