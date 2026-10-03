# {{PROJECT_NAME}} — Documentation Index

{{PROJECT_SUMMARY}}

This folder is the **contract**. Code follows these documents, never the other way around.

---

## How this repo is built

**Document-driven development.** Every feature exists as one Markdown file in `features/`
before any code is written for it. Each feature document carries its own slice of the
**validation contract**: numbered assertions that define what "correct" means, written *before*
implementation so the tests cannot simply confirm whatever the code happened to do.

The build loop is strictly **one document at a time, in order**:

```
pick next doc  ->  implement it  ->  unit tests  ->  integration tests
      ^                                                     |
      |                                                     v
      +------  mark assertions PASS  <-  live run against the real system
```

A document is not finished until every assertion in it passes against the running system.
See `foundation/05-testing-and-dod.md` for the four gates and `foundation/06-validation-contract.md`
for how assertions are numbered and tracked.

Rules that hold across every document:

- **No document, no code.** About to build something with no document? Stop and write the document. A hook enforces this.
- **One document is one unit of work.** Do not start the next while the current one has failing assertions.
- **Serial, not parallel.** One feature at a time. Only read-only work (search, research, review) runs in parallel.

---

## Foundation documents

The vocabulary and rules every feature document depends on. Written and reviewed **before** any
feature document, because an error here propagates into everything.

| # | Document | What it fixes |
|---|---|---|
| 00 | [Product specification](foundation/00-product-spec.md) | What the product is, who it is for, what it must do |
| 01 | [Tech stack](foundation/01-tech-stack.md) | Languages, frameworks, services, versions |
| 02 | [Architecture](foundation/02-architecture.md) | Layers, boundaries, and what may depend on what |
| 03 | [Domain model](foundation/03-domain-model.md) | The single source of vocabulary: entities, fields, states |
| 04 | [Interface standards](foundation/04-interface-standards.md) | API/CLI conventions, errors, versioning |
| 05 | [Testing and Definition of Done](foundation/05-testing-and-dod.md) | The four gates every feature must pass |
| 06 | [Validation contract](foundation/06-validation-contract.md) | Assertion IDs, rules, and the status ledger |
| 07 | [Security](foundation/07-security.md) | Authentication, authorization, secrets, data protection |
| 08 | [Agent harness](foundation/08-agent-harness.md) | How the project is built: roles, handoffs, hooks, memory |

---

## Feature documents — build order

Implement top to bottom. The order is a dependency order: nothing depends on a document below it.
Start each one from [`features/_TEMPLATE.md`](features/_TEMPLATE.md).

### Milestone 1 — {{MILESTONE_1_NAME}}

{{MILESTONE_1_GOAL}}

| # | Document | Depends on |
|---|---|---|
| F01 | *(not yet written)* | — |

---

## Document status

Tracked in [`foundation/06-validation-contract.md`](foundation/06-validation-contract.md). A feature is one of:

- `NOT STARTED`: document written, no code
- `IN PROGRESS`: code being written, assertions not all passing
- `DONE`: every assertion passes against the running system

No feature reaches `DONE` on unit tests alone. The live run is required.
