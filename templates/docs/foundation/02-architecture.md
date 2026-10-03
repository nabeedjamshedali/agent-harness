# 02 — Architecture

> The layers, the boundaries between them, and what may depend on what. Depends on
> `01-tech-stack.md`.

---

## 1. Principles

*The few rules that shape every design decision.*

## 2. Layers

Dependencies point **one way only**. A layer may depend on the layers below it and never on those
above.

```
   ┌──────────────────────────────┐
   │  *top layer*                 │
   └──────────────────────────────┘
   ┌──────────────────────────────┐
   │  *middle layer*              │
   └──────────────────────────────┘
   ┌──────────────────────────────┐
   │  *bottom layer*              │
   └──────────────────────────────┘
```

## 3. Request lifecycle

*One concrete request traced through every layer, showing what each does with it.*

## 4. Boundaries

*Each rule about what a layer must never know or do. These are enforced: every rule here should
also appear in `.claude/harness.json` under `boundaries`, so the `boundary_guard` hook blocks
violations at edit time, and the `boundary-auditor` checks the whole codebase.*

| Rule | Guarded layer | Forbidden | Fix when tempted to break it |
|------|---------------|-----------|------------------------------|
| *e.g. the domain is pure* | *domain/* | *I/O, framework imports* | *pass plain data in from an adapter* |

## 5. Failure and degradation

*What the system does when a dependency is slow, down or returns partial data.*
