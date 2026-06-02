## Chunk Plan

### Chunk 1 — Token store contract rewrite
- **Files:** `auth/token_store.ts`
- **Intent:** replace a generic update path with a dedicated token replacement contract
- **Why separate:** contract changes at the storage layer determine whether the rest of the PR is actually safe
- **Primary lenses:** correctness, security, API/contract compatibility
- **Potential failure modes:** partial replacement, old token remaining valid, unhandled storage failure, callers assuming old helper semantics
- **Review order:** 1

### Chunk 2 — Refresh flow behavior change
- **Files:** `auth/refresh.ts`
- **Intent:** eagerly rotate refresh tokens and use the new replacement helper
- **Why separate:** this is the main behavioral fix promised by the PR
- **Primary lenses:** correctness, security, testing
- **Potential failure modes:** replay window, non-atomic issuance/persistence, concurrent refresh edge cases
- **Review order:** 2

### Chunk 3 — Session middleware integration
- **Files:** `server/middleware/session.ts`, `server/app.ts`
- **Intent:** simplify session lookup and align middleware with new token/session handling
- **Why separate:** lower-level auth changes often look safe locally but break integration semantics at middleware boundaries
- **Primary lenses:** correctness, compatibility, observability
- **Potential failure modes:** changed null/throw behavior, different middleware ordering, silent auth regressions
- **Review order:** 3

### Chunk 4 — Test updates
- **Files:** `tests/auth/refresh.test.ts`, `tests/server/session-middleware.test.ts`
- **Intent:** update tests to reflect the new contract and validate the bugfix
- **Why separate:** tests should validate the promised guarantee, not only mirror the implementation
- **Primary lenses:** testing, correctness
- **Potential failure modes:** happy-path-only coverage, no race/failure-path checks, hidden contract drift
- **Review order:** 4
