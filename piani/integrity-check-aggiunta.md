# Testo da aggiungere al controllo di integrità mensile (attività ricorrente di ChatGPT)

Da incollare nelle istruzioni dell'attività ricorrente di ChatGPT che scrive il report in 🩺 Integrity Check. Adozione del registro condiviso: 28/09/2026.

---

**Registro condiviso (dal 28/09/2026).** Oltre ai controlli esistenti, verifica:

1. **Riferimento mancante.** Nel database 🕘 Changelog, ogni riga con `Creato` dal 28/09/2026 in poi deve avere il campo `Riferimento` non vuoto. Elenca quelle vuote (nome, data, sessione).
2. **Debiti "non committato".** Elenca le righe di Changelog il cui `Riferimento` contiene "non committato" e che non sono saldate da una riga successiva dello stesso sotto-progetto il cui `Riferimento` dice "salda il non committato del <data>". Per ognuna: nome, data, sessione, giorni trascorsi.
3. **Prese in carico appese.** Nel database 📋 Attività, elenca le righe con `Stato` = In corso, `Sessione` compilata e `Dove` compilato, per cui nel Changelog non c'è nessuna riga con la stessa `Sessione` negli ultimi 3 giorni. Per ognuna: nome dell'Attività, `Sessione`, `Dove`.

Ogni elemento trovato conta come anomalia aperta. Se tutti e tre gli elenchi sono vuoti, scrivi "Registro condiviso: nessuna anomalia".
