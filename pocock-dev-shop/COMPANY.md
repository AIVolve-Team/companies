---
name: Pocock Dev Shop
description: Executor alternativo a `.sandcastle/` che implementa ticket già decisi da `to-tickets`, usando `tdd` e `code-review` di mattpocock/skills come strumenti dei ruoli sotto forma di skill Paperclip.
slug: pocock-dev-shop
schema: agentcompanies/v1
version: 0.1.0
license: MIT
authors:
  - name: Simone
goals:
  - Eseguire in autonomia la fase implement→review→merge→qa di un batch di ticket già pianificato da to-tickets
  - Restare un executor alternativo, non un sostituto della catena wayfinder→grilling→to-spec→to-tickets (G4, G1) — quella resta manuale, fuori da qui
  - Isolare il codice fra run paralleli (worktree per branch), non solo l'assegnazione della issue
---

Pocock Dev Shop è un banco di lavoro personale: prende in carico un batch di sub-issue già
decise altrove e le porta a una PR pronta per l'Human merge, senza mai decidere *cosa*
costruire — solo *come* eseguirlo.

## Org chart

```
                 CTO (radice, nessun reportsTo)
                  |
      +-----------+-----------+-----------+
      |           |           |           |
Staff Engineer  Code Reviewer  Release Engineer  QA Engineer
```

5 agenti, tutti flat sotto CTO (nessun CEO — deciso in G2 amendment 2: i permessi che
`role: ceo` sbloccherebbe, export/import company e bypass di creazione agenti, non servono
qui perché l'export verso `paperclipai/companies` resta un atto umano).

## How Work Flows

1. **CTO** riceve il batch quando atterra su Paperclip (punto di ingresso). Rileva
   dipendenze implicite che il tracker non conosce, sceglie il batch del giro, assegna
   branch deterministici e decompone un ticket già assegnato in sub-task paralleli se serve.
2. **Staff Engineer** implementa una sub-issue sul branch assegnato, in una worktree isolata.
   Usa `tdd` per il ciclo red-green-refactor.
3. **Code Reviewer** rientra sullo stesso branch dopo Staff Engineer. Usa `code-review` e
   **corregge in loco** — nessun rimando a Staff Engineer per un semplice reject.
4. **Release Engineer** fa merge incrementale del branch (appena pronto, un branch alla
   volta) nel branch di batch — non "integration branch": nome deliberatamente diverso da
   quello già usato in `.sandcastle/` per un meccanismo opposto (CONTEXT.md righe 194-200).
5. **QA Engineer** fa da gate sul branch di batch (typecheck / test / e2e).
   - Se passa → torna a **Release Engineer**, che fa push e apre/aggiorna la PR verso il
     branch base, elencando le issue chiuse. Da lì la mano passa all'umano (Human merge).
   - Se fallisce → rimanda a **Staff Engineer**, col proprio report come contesto.

Fuori da questa company: tutta la catena a monte (`wayfinder`, `grilling`, `to-spec`,
`to-tickets`) e `triage` — girano manualmente, in un terminale Claude Code vero, perché
nessuna configurazione Paperclip invoca nativamente una skill `disable-model-invocation:
true` (G4). I ticket arrivano qui già decisi.

---

Prototipo — non genera nessuna company reale, non si importa. Scritto per il ticket P1
della mappa wayfinder #169 e criticato in sessione con Simone.
