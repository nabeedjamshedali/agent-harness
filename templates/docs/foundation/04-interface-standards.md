# 04 — Interface Standards

> Conventions for every public interface: API endpoints, CLI commands or library functions.
> Depends on `03-domain-model.md`.

---

## 1. Naming

*How endpoints, commands and parameters are named.*

## 2. Request and response shape

*The standard envelope or return shape, so every interface looks the same to its caller.*

## 3. Errors

*The error shape, the list of error codes, and which status or exit code each maps to.*

| Code | Meaning | Status / exit code |
|------|---------|--------------------|
| *not_found* | *...* | *...* |

## 4. Pagination, filtering and limits

*How large results are paged, filtered and capped.*

## 5. Versioning and compatibility

*How breaking changes are introduced, and what is guaranteed never to break.*
