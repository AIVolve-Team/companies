# Scheletro finto — cosa è saltato fuori guardandolo (P1, mappa #169)

Prototipo usa-e-getta, non una company reale. Vedi `COMPANY.md`, `.paperclip.yaml`,
`agents/*/AGENTS.md`, `skills/*/SKILL.md`.

## agents/ — una riga a testa

- `cto/` — radice, nessun reportsTo, entry point + scheduler dinamico + decompositore. **skills: [] — gap**
- `staff-engineer/` — implementa sul branch assegnato, worktree isolata. skills: [tdd]
- `code-reviewer/` — corregge in loco sullo stesso branch. skills: [code-review]
- `release-engineer/` — merge incrementale + push + PR. **skills: [] — gap**
- `qa-engineer/` — gate typecheck/test/e2e, unico reject-loop. **skills: [] — gap**

## skills/ — una riga a testa

- `tdd/` — referenziata da Staff Engineer, `usage: referenced`, commit da pinnare
- `code-review/` — referenziata da Code Reviewer, `usage: referenced`, commit da pinnare
- *(nessuna terza skill)* — `implement` **non** compare come skill referenziata: è
  user-invoked (R5), quindi resta contenuto del prompt di fase (G5) di Staff Engineer, non
  una entry in `skills:`. La skill di colla `paperclip` (parla con l'API, R4) è infrastruttura
  dell'adapter, non risulta negli esempi reali come skill dichiarata in `AGENTS.md` — da
  confermare, non l'ho verificato sul catalogo.

## Cose che sulla pagina non tornano (da criticare)

1. **Tre ruoli su cinque non hanno nessuna skill Pocock** (CTO, Release Engineer,
   QA Engineer). A parole sembrava che "le skill si partizionano fra i ruoli" bastasse come
   principio; sulla carta si vede che metà organico non ha uno strumento importato — fanno
   tutto per prosa nel prompt di fase. È voluto (nessuna skill del catalogo mattpocock/skills
   copre scheduling, git-merge-e-PR, o gate QA) o manca qualcosa che nessun ticket ha
   guardato? `resolving-merge-conflicts` per Release Engineer è l'unico candidato non ancora
   valutato in nessun ticket chiuso.

2. **Il ticket P1 stesso presupponeva che `.paperclip.yaml` porti "agenti e gerarchia"** — R1
   ha già stabilito il contrario (hierarchy vive in `reportsTo` dentro `AGENTS.md`, il sidecar
   è solo adapter/modello/env), ma vederlo scritto nella issue originale e poi smentito sulla
   carta è la wrapper mismatch più diretta che il prototipo doveva far emergere.

3. **`isolated_workspace` / `strategy: git_worktree` (R6) non ha una casa ovvia nello
   scheletro.** Non è un campo di `.paperclip.yaml` (è impostazione di progetto Paperclip, non
   del company package) — quindi S1 deve descriverla come passo di **setup del progetto**,
   separato dai file versionabili della company. Nessun ticket chiuso l'aveva reso esplicito
   fin qui.

4. **`AGENTS.md` qui non ha ancora i quattro paragrafi prescritti** (da dove arriva il
   lavoro / cosa produci / a chi passi / cosa ti attiva, R1 §3) né il prompt di fase — per
   scelta: G5 li assegna al file di prompt colocato (opzione 3), non ancora scritto, e in
   attesa che R8 verifichi che Paperclip carichi file oltre `AGENTS.md` dalla cartella agente.
   Se R8 falsifica, questi cinque file diventano il posto dove tutto quel testo rientra
   inline (opzione 2, fallback già deciso in G5).

5. **Nessun nome di file deciso per il prompt colocato** (opzione 3, G5) — `plan-prompt.md`
   / `implement-prompt.md` / `review-prompt.md` / `merge-prompt.md` (nomi reali sandcastle) o
   qualcos'altro? Non ticketato da nessuna parte, R8 lo sblocca ma non lo risponde.

## Cosa NON c'è in questo scheletro (deliberatamente)

- Nessun `README.md` / `LICENSE` / `images/org-chart.png` — decorativi per questo esercizio.
- Nessun prompt di fase reale nei cinque `AGENTS.md` — è lavoro di S1/G5, non di P1.
- Nessun commit SHA vero nei due `SKILL.md` — richiede clonare `mattpocock/skills` e pinnare,
  fuori scope di un prototipo di carta.
