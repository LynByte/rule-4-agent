# Agent operating contract — bootstrap template

Purpose: drop this file into the root of ANY project. The agent working in that
project reads it first, inspects the repository, then generates a project-
specific `AGENTS.md` plus `.agents/rules/*` and `.agents/skills/*` tailored to
that codebase. After generation, this template can be removed — the generated
files are the durable contract.

The agent MUST NOT copy this file verbatim. Every command, path, module name,
and risk statement in the generated rules must be discovered from the actual
repository, not guessed.

---

## Part 1 — Non-negotiable principles (apply to every project, unchanged)

These are the invariant rules. They belong in every generated `00-core` rule
file and always take precedence over convenience.

1. Preserve user data, persisted state compatibility, and unrelated worktree
   changes. Stage only the task's own files; never touch unrelated dirty
   changes.
2. Read the narrowest relevant files before editing. Reuse existing patterns;
   do not create a parallel state, service, or cache path without a reason.
3. Make focused, reviewable changes. Separate mechanical cleanup from behavior
   changes unless the request needs both.
4. Prefer explicit types, guards, and invariants. Never solve a correctness
   issue with global lint disables, broad casts, or swallowed errors.
5. Manage every owned resource: event listeners, timers, RAF callbacks,
   workers, object/blob URLs, subscriptions, streams, sockets, file handles.
   Release them on every complete, cancel, error, and unmount path.
6. Do not expose secrets, log sensitive data, or commit credentials.
7. Fix the root cause, not the symptom. Trace the causal chain before editing.
   Hiding an error, special-casing one call site, suppressing a check, or
   papering over nulls while the defect remains is forbidden.
8. Weigh trade-offs and hold production-grade correctness: concurrent access,
   rename/delete lifecycle, authorization boundaries, stale references, scale
   limits, and backward compatibility. State the alternatives considered and
   why the chosen one wins.
9. Never perform irreversible or externally-visible actions without explicit
   user approval: deleting data, force-push, history rewrite, production
   migrations/deploys, sending messages, payments, third-party API mutations.
10. State assumptions and verification limits clearly in the handoff.

## Part 2 — Delivery rules (apply to every project, unchanged)

1. A task that creates, modifies, or deletes project files is not complete
   until proportional verification passes and its scoped changes are in one
   focused local commit. Stage task paths explicitly, inspect the staged diff,
   run `git diff --check` (or the platform equivalent), then commit.
   Read-only investigations require no commit.
2. Keep commits cohesive and reversible. Do not include unrelated user
   changes unless the user explicitly asks for all pending work.
3. Do not push by default. Push only when the user explicitly asks to push
   the current branch or names a target remote/branch.
4. Report the commit hash, branch, verification performed, and any
   intentionally uncommitted files.
5. If a safe commit cannot be created, report the blocker and treat the task
   as incomplete — never leave completed work silently uncommitted.

## Part 3 — Verification baseline (generated per project)

Verification commands are discovered, not templated. During bootstrap, the
agent must locate the project's real equivalents of:

- **Formatter/linter**: e.g. `biome check`, `eslint`, `prettier`, `ruff`,
  `cargo clippy`, `gofmt` — read `package.json` scripts, `biome.json`,
  `.eslintrc*`, `pyproject.toml`, `Cargo.toml`, CI workflows.
- **Typecheck**: `tsc --noEmit`, `mypy`, `cargo check`, `go vet`.
- **Tests**: the project's test runner and the closest existing test files.
- **Build**: the production build command.
- **CI gates**: read `.github/workflows/*` (or the platform's CI config) and
  reference the actual gate names.

The generated `AGENTS.md` must name the exact commands. If none can be
determined, write the fallback ("ask the user before handoff") rather than
inventing a command.

## Part 4 — Bootstrap protocol

When this template is present in a repo and no `AGENTS.md` (or equivalent
agent contract) exists yet, the agent must:

1. **Map the repo.** Identify: language(s), framework(s), package manager,
   monorepo vs single app, module boundaries, build/test/lint tooling, CI.
2. **Classify risk surfaces.** Find: user-data persistence, databases and
   their environments, secrets handling, external paid APIs, destructive
   operations, hot paths (rendering, realtime, per-event work), and any
   resource types needing explicit cleanup (media, workers, sockets).
3. **Generate `AGENTS.md`** at the repo root using the skeleton in Part 5.
   Fill every placeholder with facts from step 1–2.
4. **Generate `.agents/rules/`**:
   - `00-core.md` — copy Part 1 + Part 2 of this template verbatim. It never
     changes per project.
   - `README.md` — routing table mirroring Part 5's rule table.
   - One additional rule file per major risk surface found in step 2
     (e.g. `15-database.md`, `20-realtime.md`, `20-media.md`, `20-billing.md`).
     Each rule file is short, imperative, and testable — like the LynCut
     `15-database-targeting.md` or `20-nle-editor.md`, but written for THIS
     project's actual risk model.
   - `30-quality.md` — verification ladder using the real commands from
     Part 3.
   - `40-delivery.md` — copy Part 2 verbatim plus any project-specific
     release gates.
5. **Generate `.agents/skills/`** only for workflows that are non-obvious and
   repeated (e.g. "how to run the focused test suite", "how to apply a DB
   migration safely"). Each skill is a folder with a `SKILL.md` using YAML
   frontmatter (`name`, `description`) — see Part 6.
6. **Report back.** List generated files, the discovered verification
   commands, and any risk surfaces where the agent could not determine the
   right rule — ask the user for those specifics instead of guessing.

If the project already has its own agent contract (`AGENTS.md`, `CLAUDE.md`,
`.cursor/rules`, etc.), do not overwrite it. Read it, note gaps against this
template, and propose additions instead.

## Part 5 — Generated `AGENTS.md` skeleton

```markdown
# <Project> agent control center

<Project> is <one-sentence: what it is, what stack>. Preserve
<top-3 things that must never break: e.g. user data, billing correctness,
render output>.

## Start here

1. Read this file and `.agents/rules/00-core.md`.
2. Choose the matching task rule below.
3. Read only the docs and files needed for the task.
4. Keep changes focused; do not mix refactor, formatting, and behavior.
5. Verify in proportion to risk and report what was checked.

## Rule routing

| Trigger | Read in addition to `00-core.md` |
| --- | --- |
| Module ownership, imports, refactor boundary | `.agents/rules/10-architecture.md` |
| <project-specific risk 1> | `.agents/rules/15-<name>.md` |
| <project-specific risk 2> | `.agents/rules/20-<name>.md` |
| Any behavioral, timing, or performance change | `.agents/rules/30-quality.md` |
| Commit, push, migration, delete, deploy, release | `.agents/rules/40-delivery.md` |

## Architecture boundaries

<2–6 bullets describing the real module map and dependency direction —
e.g. "UI → features → core → services" or "app/ → packages/ → infra/".
Lower layers must not import upper layers.>

## Non-negotiable safety rules

<Copy the 10 core principles from Part 1 verbatim, plus any project-specific
invariants discovered in bootstrap step 2 — e.g. "never edit generated
files", "all DB writes go through the repository layer".>

## Verification baseline

- <Change type A>: <real lint + typecheck + closest test commands>
- <Change type B>: <real gate commands>
- Database change: <real migration/seed procedure or "ask first">
- A file-changing task is complete only after proportional verification and
  one focused local commit. Do not push by default.
```

## Part 6 — Skill format

`.agents/skills/<skill-name>/SKILL.md`:

```markdown
---
name: <skill-name>
description: <one line — when to use this skill, what it verifies/does>
---

# <Title>

<Concise imperative steps. Reference real commands, real file paths,
and the project's actual gates. No filler.>
```

Generate a skill only when a workflow is (a) repeated and (b) has enough
project-specific detail that re-discovering it each session is error-prone.
Do not generate skills for one-off tasks.

## Part 7 — What must never be copied

- Verification commands from another project (e.g. `bunx tsc -p apps/web/
  tsconfig.json` belongs to LynCut, not to this project).
- Module maps, port numbers, DB project refs, endpoint names, table owners.
- Domain rules written for another product (media timing rules, billing
  rules) unless the domain genuinely matches.
- CI gate names — always read this repo's own CI config.

If the agent cannot determine a fact, it leaves an explicit `<TODO: confirm
with user>` in the generated file and asks — it does not invent values.
