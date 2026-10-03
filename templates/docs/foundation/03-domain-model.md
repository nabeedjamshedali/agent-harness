# 03 — Domain Model

> **The single source of vocabulary.** Every entity, field and state the project talks about is
> defined here, once. Feature documents and code use these names exactly. Depends on
> `02-architecture.md`.

---

## 1. Entities

*For each entity: what it represents, its fields with types, and which are optional.*

```
Entity
    id: ...
    field: type          # meaning
    optional_field: type | None   # None means "not available", never "zero"
```

## 2. States and enums

*Every fixed set of values (statuses, roles, kinds), with what each means.*

## 3. Relationships

*How entities refer to each other, and what happens to dependents when one is removed.*

## 4. Missing versus empty

*For each optional field: what "not available" means, what "empty/zero" means, and how the user is
told the difference. Conflating them produces confident wrong answers.*

## 5. Changing the model

A feature that needs a field which is not here does not invent it. It raises it as an open question,
the field is added here first, then migrated, then used.
