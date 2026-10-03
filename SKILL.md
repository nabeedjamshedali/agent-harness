---
name: agent-harness
description: Install a document-driven, multi-agent build harness into the current project - an orchestrator, one worker per architectural layer, four adversarial validators, four enforcing hooks (no code without a feature document, architectural boundary guard, assertion coverage, no finish without a handoff), a .memory/ folder for missions, handoffs and decisions, and a docs/ contract with numbered validation assertions and a four-gate Definition of Done. Works on new and existing projects in any language. Use when the user runs /agent-harness or asks to set up the agent harness, missions workflow or document-driven development in a project.
argument-hint: "[optional one-line description of the project]"
disable-model-invocation: true
---

# Agent Harness

You are installing a build harness that makes long, multi-session agent builds stay correct.
Its core rules, which every file you install enforces:

1. **No document, no code.** Every feature is a document in `docs/features/` with numbered
   assertions (`F07-A03`) written *before* implementation.
2. **One feature at a time.** An orchestrator plans; one worker implements; validators verify.
3. **Validators never trust the author.** Four independent checks, none reading the code first.
4. **Done means proven on the running system**, not "unit tests green".
5. **Nothing is lost between sessions.** Handoffs, decisions and mission state live in `.memory/`.

Templates live in `templates/` inside this skill's base directory (shown when the skill loads).
User argument, if any: `$ARGUMENTS`

## Ground rules for the install

- **Never overwrite an existing file.** Create only what is missing. If a target already exists,
  leave it, and list it in your report as "kept existing". The one exception is
  `.claude/settings.json`, which you **merge** into (see step 4).
- **Never touch application source code.** You install harness files only.
- **Every value you write must come from the real project.** Read before you write; do not guess
  paths or commands you have not seen.
- If `.claude/harness.json` already exists, this is a **re-run**: only add missing files, then show
  the user which harness files differ from the templates and ask before updating any of them.

## Step 1 — Survey the project (read-only)

Find the project root (`git rev-parse --show-toplevel`, else the working directory). Then learn:

- **Name and purpose**: README, package manifest, `$ARGUMENTS`.
- **Stack**: manifests and lock files (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`,
  `pom.xml`, `*.csproj`, `composer.json`, `Gemfile`, ...).
- **Source paths**: where implementation code lives (`src/`, `app/`, `backend/src/...`, `lib/`,
  `internal/`, `pkg/`, ...). These become `code_paths`.
- **Test paths and naming**: where tests live and their file pattern. These become `test_paths`.
- **Commands**: unit tests, full suite, lint/format/type-check, and how to boot the system
  (manifest scripts, Makefile, `docker-compose.yml`, CI config).
- **Layers**: the top-level folders inside the source paths that are distinct architectural
  layers (e.g. `api`, `domain`, `services`, `adapters`, `ui`). Each becomes one worker.
- **Existing harness pieces**: `.claude/`, `.memory/`, `docs/`.

**Greenfield project** (no source code yet): ask the user, in one `AskUserQuestion` call, what they
are building, the stack, and the layers they expect. Propose sensible defaults for that stack as
the first option of each question.

**Existing project**: do not interview. Infer everything above. Ask only if two readings would
produce materially different harnesses (for example, you cannot tell which folders are layers).

## Step 2 — Decide the configuration

Work out, and keep for the next steps:

| Placeholder | Value |
|---|---|
| `{{PROJECT_NAME}}` | The project's name |
| `{{PROJECT_SUMMARY}}` | One or two sentences on what it is and who it is for |
| `{{MISSION_GOAL}}` | The first milestone, as a goal sentence |
| `{{MILESTONE_1_NAME}}` / `{{MILESTONE_1_GOAL}}` | The first milestone's name and its goal |
| `{{RUN_COMMAND}}` | How to boot the real system |
| `{{UNIT_TEST_COMMAND}}` / `{{TEST_COMMAND}}` | Unit tests only / the full suite |
| `{{LINT_COMMAND}}` | Lint, format and type-check |
| `{{PYTHON}}` | The Python interpreter for hooks (see below) |
| `{{WORKER_TABLE}}` | Markdown table: Worker / Owns / Key constraint, one row per worker incl. `doc-worker` |

Where a command does not exist yet, write `TBD — define in 01-tech-stack.md`; never invent one.

**Workers.** One per layer, named `<layer>-worker` (e.g. `api-worker`, `domain-worker`), plus the
fixed `doc-worker`. Give each a concrete, layer-specific constraint list: what it may depend on,
what it must never do, and the conventions of that layer in this stack.

**Boundaries.** For each dependency rule implied by the layers (a pure core, a layer that must not
know a framework, an inner layer that must not import an outer one), write one entry in
`harness.json` → `boundaries`. Use `templates/harness.example.json` as the shape: `guarded_paths`,
`extensions`, `forbidden` regex patterns each with a `reason`, and a `fix`. Patterns are Python
regexes, matched per line with MULTILINE. Prefer a few precise rules over many fuzzy ones: a rule
that fires on harmless code trains people to ignore it. If the architecture has no clear
boundary yet, leave `boundaries` empty and say so in the report.

**Python for hooks.** The hooks are standard-library Python 3.8+. Find a working interpreter by
running, in order, `python3 -c "import sys; print(sys.version_info >= (3, 8))"` then the same with
`python`, and use the first that prints `True`. (On Windows, `python3` is often a Store stub that
prints nothing.) If neither works, stop and tell the user Python 3.8+ is required for the hooks;
everything else can still be installed.

## Step 3 — Install the files

Copy from `templates/`, replacing every placeholder:

| Template | Destination |
|---|---|
| `hooks/*.py` (all five, including `_harness.py`) | `.claude/hooks/` |
| `agents/orchestrator/mission-orchestrator.md` | `.claude/agents/orchestrator/` |
| `agents/validators/*.md` | `.claude/agents/validators/` |
| `agents/workers/doc-worker.md` | `.claude/agents/workers/` |
| `agents/workers/_worker-template.md` | `.claude/agents/workers/<layer>-worker.md`, once per layer, filling `{{WORKER_NAME}}`, `{{LAYER}}`, `{{PATHS}}`, `{{CONSTRAINTS}}`. Do not copy the template itself. |
| `memory/**` | `.memory/**` (keep `handoffs/_TEMPLATE.md` and `decisions/_TEMPLATE.md`) |
| `docs/README.md`, `docs/features/_TEMPLATE.md`, `docs/foundation/*.md` | `docs/` (same sub-paths) |
| *(your decisions from step 2)* | `.claude/harness.json` |

If the project has no layered architecture worth guarding, still install `boundary-auditor.md`; it
is harmless with no rules and useful once rules exist.

Write `harness.json` with `enabled`, `features_dir`, `handoffs_dir`, `code_paths`, `code_exempt`,
`test_paths`, and `boundaries`. Put entry points, config, migrations, generated code and test
helpers in `code_exempt` (substring matches against the repo-relative path, e.g. `"/migrations/"`).
Write JSON with a real serializer (for example `python -c "import json; ..."`), not by hand-escaping
regexes in a shell, which silently corrupts backslashes.

## Step 4 — Wire the hooks

Merge `templates/settings.hooks.json` (with `{{PYTHON}}` filled in) into `.claude/settings.json`:

- No settings file: create it with exactly that content.
- Existing file: parse it, append each hook group to the matching event list (`PreToolUse`,
  `PostToolUse`, `Stop`), keep every other key untouched, and skip any hook whose command already
  references the same script. Write it back with a JSON serializer.

Tell the user to review and approve them: hooks run shell commands, and Claude Code shows new hooks
for approval (run `/hooks` to see them).

## Step 5 — Verify before reporting

Run each check and fix what fails. Report the real results.

1. `python -m json.tool` succeeds on `.claude/harness.json`, `.claude/settings.json` and
   `.memory/contracts/ledger.json`.
2. Every hook compiles: `{{PYTHON}} -m py_compile .claude/hooks/*.py`.
3. No placeholder is left: search the installed files for `{{`. `F{NN}`-style placeholders inside
   the two `_TEMPLATE.md` files are intentional; everything else must be filled.
4. Probe the hooks with real payloads from the project root. Use a path that does **not** exist, so
   nothing is created:
   - `echo '{"tool_input":{"file_path":"<root>/<first code path>/__harness_probe__.ext"}}' | {{PYTHON}} .claude/hooks/doc_exists.py` → exit **2** while no feature document has assertions (that is the guard working), 0 afterwards.
   - `echo '{"stop_hook_active": true}' | {{PYTHON}} .claude/hooks/handoff_guard.py` → exit **0**.
   - `echo '{}' | {{PYTHON}} .claude/hooks/boundary_guard.py` → exit **0**, and no `harness.json is invalid` warning in its output.

## Step 6 — Report

Tell the user, briefly:

- **Installed**: the counts (agents, hooks, docs, memory files), the workers you created and the
  layer each owns, and the boundary rules in one line each.
- **Kept existing**: any file you did not overwrite.
- **Verification**: the step 5 results.
- **What happens now**: the hooks are active once approved. Creating a new source file is
  **blocked until a feature document with assertions exists**; that is intended. Editing existing
  files is never blocked. To pause all four hooks, set `"enabled": false` in `.claude/harness.json`.
- **Next step**: ask the `mission-orchestrator` agent to start. It will have `doc-worker` write the
  foundation documents (00 product spec first), then the first feature document, then dispatch the
  first worker.

Keep the report short. The files explain themselves; the user needs to know what changed, what was
verified, and what to do next.
