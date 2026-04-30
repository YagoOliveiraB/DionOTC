# Roadmap Codex - Portable Roadmap Policy

## Purpose
This file is the entry point to start and maintain a complete roadmap for any large topic (feature, migration, broad refactor, integration, etc.).

Use this document when you want to ask:
- "create a roadmap for X"
- "structure milestones and tasks for Y"
- "replan because one milestone is blocking another"

## Invocation
Recommended prompt reference:
- `@docs/roadmap_codex.md`

Accepted alias:
- `@docs/roadmap.md` (if it exists as a pointer to this document)

## Output Contract (always)
When starting a new roadmap, the agent must create/update in `docs/roadmap/`:
- `agents.md`
- `vision.md`
- `roadmap.md`
- `milestones.md`
- `tasks.md`
- `status.md`
- `decisions.md`
- `architecture.md`

## Bootstrap Rules
1. Check whether `docs/roadmap/` exists.
2. If it does not exist, create the folder and the 8 canonical files.
3. Check whether `docs/roadmap` is ignored in `.gitignore`.
4. If it is not ignored and the context is local planning, add it under:
   - section `# Projects IA`
   - entry: `docs/roadmap`
5. Never delete an existing roadmap; perform incremental merges.

## Guides Reference Protocol
Before proposing milestones/tasks, the agent should evaluate reusable standards in `docs/guides/`:
- `docs/guides/cpp_guide.md`
- `docs/guides/lua_guide.md`
- `docs/guides/git_guide.md`

Rules:
1. Reference applicable guide(s) in `docs/roadmap/agents.md` for the active subject.
2. If a guide conflicts with current repository style, prefer repository style and register the decision in `docs/roadmap/decisions.md`.
3. Keep `docs/guides/*.md` referenced in `wiki.md` so they are included by `tools/export_wiki_bundle.py`.
4. Do not duplicate entire guides inside roadmap files; add only targeted references.

## Scope Sizing (dynamic milestones)
Define initial size based on risk + coupling + impact:

- Small (1-2 milestones)
  - Localized change, low risk, few files.
- Medium (3-5 milestones)
  - Multiple modules, partial integration, moderate manual validation.
- Large (6+ milestones)
  - Cross-cutting change, protocol/API contract changes, external dependency, high regression risk.

Rule:
- Start with the smallest viable number.
- Split milestones when there is a blocker or mixed scope.
- Merge milestones when overlap is high and risk is low.

## Dependency and Blocking Model
Each milestone must declare:
- `depends_on`: prerequisite milestones/tasks
- `blocks`: which items are blocked if it fails
- `unblock_strategy`: minimum plan to unblock

Each task must declare:
- Unique ID (`Tx.y`)
- Direct dependencies
- Verifiable acceptance criteria
- Expected evidence (test, log, build, capture)

## State Machine (roadmap flow)
Valid milestone states:
- `pending`
- `in_progress`
- `blocked`
- `at_risk`
- `completed`

Transitions:
- `pending -> in_progress`: when `depends_on` is satisfied.
- `in_progress -> blocked`: external dependency or structural failure.
- `blocked -> in_progress`: blocker removed with evidence.
- `in_progress -> at_risk`: progress exists, but delivery confidence is low.
- `at_risk -> in_progress`: mitigation plan applied.
- `in_progress -> completed`: exit criteria validated.

Controlled rollback:
- If a milestone fails and affects others, move dependents to `blocked`.
- Replan in `milestones.md` and `tasks.md` without losing history.

## Required Quality Rules
- Do not create tasks outside `tasks.md` without registering them first.
- Do not change architecture without recording it in `decisions.md`.
- Prefer tasks that touch 1-3 files when feasible.
- Every item in `status.md` must point to a real task.
- Every completed milestone must have minimum evidence.

## Git Policy (from global context)
- Commits must be atomic.
- Use commit prefixes by type:
  - `fix:` bug fix
  - `feat:` new feature
  - `imp:` improvement/refactor without a new feature
  - `docs:` documentation
- Avoid mixed commits (code + docs + config) when possible.
- Keep commit messages short and objective.

## Agent-Only Artifacts Policy
- Agent operational artifacts (execution planning, local status, local technical backlog) must live in `docs/roadmap/`.
- `docs/roadmap/` should remain git-ignored when used for local planning.
- Because of this rule, a dedicated commit prefix like `codex:` is not required for these artifacts.
- When documentation must be versioned in the repository, use `docs:`.

## Commit Gate (mandatory for all commit types)
Before any commit (including `feat`, `imp`, `docs`), the agent must:
1. Ask for explicit user confirmation before committing.
2. Confirm the user had a chance to test (or accepted committing without testing).
3. Record in the commit summary whether it was:
   - `tested by user`, or
   - `not tested by explicit user decision`.

Without explicit confirmation, do not commit.

## Mandatory Execution Loop
1. Read `status.md`
2. Execute `NEXT_TASK`
3. Validate minimum viable scope
4. Update `status.md`
5. Sync `tasks.md`, `milestones.md`, and `decisions.md` if the plan changed

## Minimum `status.md` Contract
```md
CURRENT_MILESTONE: Mx - Name
NEXT_TASK: Tx.y - Task name

BLOCKERS:
- none

RECENT_CHANGES:
- YYYY-MM-DD - objective summary
```

## When to add more milestones
Add a milestone when:
- there is a strong external dependency;
- there is a contract change (API/protocol/schema);
- validation needs a dedicated phase;
- regression risk justifies a dedicated hardening phase.

## When to go back to a previous milestone
Move back when:
- an exit criterion was broken by regression;
- a late-discovered dependency invalidates current work;
- validation evidence contradicts the architecture assumption.

## Reuse Template (new subject)
Suggested base prompt:

```md
@docs/roadmap_codex.md
Create/update a roadmap for: <subject>
Context: <affected modules, constraints, risks>
Final goal: <functional outcome>
```

## Notes
- This file is intentionally generic and reusable.
- The resulting roadmap must be specific to the current subject.
