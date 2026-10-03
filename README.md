# agent-harness

**A Claude Code skill that installs a document-driven, multi-agent build harness into any project with one command.**

```
/agent-harness
```

Claude reads your codebase, works out its layers, and installs an orchestrator, one worker per
layer, four independent validators, four enforcing hooks, a memory folder and a documentation
contract, all shaped to your project. From then on, the rules you keep repeating to Claude are
enforced by the harness instead of by you.

---

## Why

Long agent builds go wrong in predictable ways:

- **Tests confirm the code instead of the requirement.** The agent writes code, then writes tests
  that describe what the code does, including its bugs. Everything is green, and it's wrong.
- **Architecture drifts.** By the thirtieth feature, a layer that was supposed to stay clean imports
  something it shouldn't. Nothing fails, so nobody notices until the fix is expensive.
- **"Done" means "unit tests pass".** Nobody booted the real system.
- **Context is lost between sessions.** Decisions made on Monday are silently reversed on Thursday,
  and unfinished work is reported as finished.
- **You become the guardrail.** You spend your time typing "you forgot the tests", "don't import
  that here", "did you actually run it?", again and again.

A harness moves that discipline out of your prompts and into the project. Deterministic hooks handle
the bookkeeping and the blocking; the model handles the judgement.

> The harness provides the discipline. The models provide the intelligence.

---

## What it installs

| | What | Why |
|---|---|---|
| **Orchestrator** | Picks the next feature, scopes it for one worker, reviews the handoff, decides whether to advance | Planning separated from doing |
| **Workers** | `doc-worker` plus one worker per layer of *your* codebase (`api-worker`, `domain-worker`, ...) | Each knows its layer's rules and touches nothing else |
| **Validators** | `scrutiny-validator`, `behaviour-validator`, `contract-auditor`, `boundary-auditor` | Verification by an agent that didn't write the code |
| **Hooks** | Four Python guards wired into Claude Code | The rules are enforced, not requested |
| **`.memory/`** | Mission state, structured handoffs, decision records, an assertion ledger | Nothing is lost between sessions |
| **`docs/`** | Foundation docs 00–08 and a feature template with numbered assertions | Code is built against a written contract |

### The four hooks

| Hook | When | What it does |
|---|---|---|
| `doc_exists` | Before a new source file is created | **Blocks** it until a feature document with numbered assertions exists. No document, no code. |
| `boundary_guard` | After a file in a guarded layer is edited | **Blocks** imports and concepts that cross your architecture's boundaries, and tells Claude how to fix the design |
| `assertion_coverage` | After a test file is edited | Reports which assertions still have no test |
| `handoff_guard` | When Claude tries to finish | **Blocks** finishing until a handoff records what was done, what was *not* done, and the commands that proved it |

Editing existing files is never blocked. Every hook fails open on its own errors, a broken config is
reported rather than silently ignored, and a second stop always goes through, so a guard can never
trap your session.

---

## How it works

Every feature is a document with numbered assertions, written **before** any code:

```
F07-A01  [U]  An expired token is rejected even when its signature is valid
F07-A02  [I]  A wrong password returns 401 and writes no session
F07-A03  [L]  Logging in through the running system, then calling a protected endpoint, succeeds
```

`[U]` unit, `[I]` integration, `[L]` live run against the real system. Tests are named after the
assertion they prove (`test_F07_A03_...`), so coverage is checked mechanically.

Then one loop, one feature at a time:

```
   ORCHESTRATOR picks the next document
        │
        ▼
   Assertions written?  ── no ──>  doc-worker writes them first
        │ yes
        ▼
   WORKER implements it, tests it, writes a handoff
        │
        ▼
   VALIDATORS: scrutiny · behaviour · contract · boundary
        │
        ├── fail ──>  precise follow-up work, then re-validate
        ▼ pass
   Ledger updated, full suite green  ──>  next document
```

A feature is **done** when its assertions pass against the running system, not when its unit
tests are green.

---

## Install

You need [Claude Code](https://claude.com/claude-code) and Python 3.8+ (the hooks use only the
standard library).

**Option 1: clone it into your skills folder**

macOS / Linux:
```bash
git clone https://github.com/nabeedjamshedali/agent-harness ~/.claude/skills/agent-harness
```

Windows (PowerShell):
```powershell
git clone https://github.com/nabeedjamshedali/agent-harness "$env:USERPROFILE\.claude\skills\agent-harness"
```

Restart Claude Code, open any project, and run `/agent-harness`.

**Option 2: let Claude install it**

Paste this into Claude Code:

> Install the agent-harness skill from https://github.com/nabeedjamshedali/agent-harness into my
> personal skills folder (~/.claude/skills/agent-harness), then follow its SKILL.md to install the
> harness in this project.

---

## Usage

```
/agent-harness
/agent-harness an invoicing API for small accounting firms
```

- **Existing project:** no questions. It infers your stack, source and test folders, commands and
  layers from the code.
- **Empty project:** it asks once what you're building, the stack and the layers.
- **Safe:** it never overwrites an existing file, never touches your application code, and merges
  into your existing `.claude/settings.json` instead of replacing it.
- **Verified:** before reporting, it checks every JSON file, compiles every hook and probes each one
  with a real payload.

Then ask the `mission-orchestrator` agent to start. It has the foundation documents written first,
then the first feature, then dispatches the first worker.

---

## Configuration

The skill writes `.claude/harness.json`, the one file the hooks read:

```json
{
  "enabled": true,
  "code_paths": ["src"],
  "test_paths": ["tests"],
  "code_exempt": ["__init__.py", "/migrations/"],
  "boundaries": [
    {
      "name": "Pure domain",
      "guarded_paths": ["src/domain"],
      "extensions": [".py"],
      "forbidden": [
        { "pattern": "^\\s*(from|import)\\s+(requests|sqlalchemy)\\b", "reason": "the domain must not perform I/O" }
      ],
      "fix": "Move the I/O into an adapter and pass plain data into the domain."
    }
  ]
}
```

- **`boundaries`**: your architecture rules, as regex patterns forbidden in guarded paths. Start
  with a few precise rules; a rule that fires on harmless code trains people to ignore it.
- **`"enabled": false`** pauses all four hooks.

---

## Works even better with superpowers

[superpowers](https://github.com/obra/superpowers) by Jesse Vincent and Prime Radiant (not part of
this project) is a set of skills that shape **how each piece of work is done**: brainstorming the
design, writing a plan, test-driven development, subagent-driven execution, systematic debugging,
code review.

agent-harness shapes **the project around that work**: what gets built next, the contract it's
built against, the boundaries it can't cross, independent validation, and memory across sessions.
They don't overlap, and they combine well: superpowers keeps each task disciplined, and the harness
keeps the whole build disciplined.

```
/plugin install superpowers@claude-plugins-official
```

---

## Credits

The methodology adapts the "Missions" architecture described by Factory: an orchestrator,
workers and validators; validation contracts written before implementation; structured handoffs;
and serial execution.
