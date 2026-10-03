# 07 — Security

> Authentication, authorization, secrets and data protection. Depends on `02-architecture.md`.

---

## 1. Authentication

*How users and services prove who they are, how sessions or tokens work, and how they expire.*

## 2. Roles and permissions

| Role | Can | Cannot |
|------|-----|--------|
| *...* | *...* | *...* |

Feature documents reference these roles rather than redefining them.

## 3. Data isolation

*If the system holds data for more than one user or account: how isolation is enforced, and what a
request for someone else's data returns.*

## 4. Secrets

*Where secrets live, how they are stored at rest, and the rule that they never appear in logs,
responses, errors or the repository.*

## 5. Audit

*Which actions are recorded, with what detail, and who can read the record.*

## 6. Threats considered

*The specific attacks this design defends against, and the ones it explicitly does not.*
