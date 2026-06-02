## Guided Review

### Chunk 1 — Token store contract rewrite

**What changed**
- The generic `tokenStore.update()` path was replaced by a more specific `replaceSessionTokens()` helper.
- Token replacement logic appears to be centralized into one storage API.

**Relevant context**
- The entire PR depends on this helper defining the correct invariants.
- If the helper does not guarantee single-use replacement semantics, the rest of the fix is mostly cosmetic.

**What to inspect**
- Does replacement invalidate the old refresh token before or atomically with writing the new one?
- What happens if persistence fails after new tokens are generated?
- Do any callers still assume `update()` semantics, especially partial field updates?

**Potential concerns**
- **Observed fact:** the storage abstraction changed from a generic update helper to a purpose-built replacement helper.
- **Inference:** the author intends to make token replacement the only valid mutation path.
- **Risk hypothesis:** if replacement is implemented as separate invalidate/write steps without a stronger atomicity guarantee, a replay window may still exist.

**Questions**
- Is `replaceSessionTokens()` intended to be atomic at the storage boundary?
- Are all previous partial-update callers migrated or intentionally removed?

**Tentative AI assessment**
- likely blocking issue unless atomicity/failure semantics are clearly safe

---

### Chunk 2 — Refresh flow behavior change

**What changed**
- Refresh handling now rotates tokens eagerly after verification and uses `replaceSessionTokens()` instead of the older update flow.

**Relevant context**
- The PR claims to fix a race where two close refresh requests could both succeed.
- That claim is only true if verification, replacement, and response issuance interact safely under concurrent requests.

**What to inspect**
- Can two concurrent refreshes both pass verification before the first write lands?
- Is the old refresh token definitely rejected after the first successful replacement?
- If replacement fails, does the handler fail closed or return a token pair that was never persisted?

**Potential concerns**
- **Observed fact:** token rotation was moved earlier and made more explicit in the refresh flow.
- **Inference:** the code is trying to reduce a stale-token success window.
- **Risk hypothesis:** if token issuance occurs before durable replacement, the system may still produce valid-looking but non-persisted tokens or allow concurrent success.

**Questions**
- Do we require strict single-use refresh semantics, or only best-effort reduction of the race?
- Is there any locking/compare-and-swap behavior at the store layer, or is correctness relying on request ordering?

**Tentative AI assessment**
- needs clarification, with a plausible blocking concurrency concern

---

### Chunk 3 — Session middleware integration

**What changed**
- Session lookup/null handling was simplified, and middleware now appears to rely on the consolidated token replacement/session path.

**Relevant context**
- Middleware changes are dangerous because many downstream handlers rely on stable behavior for missing or invalid session state.
- A cleanup that changes throw/null behavior can become a broad compatibility regression.

**What to inspect**
- Did missing session behavior stay the same?
- Does invalid token handling still map to the same downstream outcome and logging?
- Was middleware ordering or app wiring changed in a way that affects unauthenticated routes?

**Potential concerns**
- **Observed fact:** null-check duplication was removed as part of the simplification.
- **Inference:** the author believes behavior is equivalent but cleaner.
- **Risk hypothesis:** behavior may no longer be equivalent for missing-session vs invalid-session cases, especially if one path now throws where it previously returned null.

**Questions**
- Are downstream handlers/tests relying on `null`-style semantics anywhere?
- Did logging/observability change for rejected session paths?

**Tentative AI assessment**
- likely non-blocking if compatibility is preserved, but worth explicit confirmation

---

### Chunk 4 — Test updates

**What changed**
- Auth and middleware tests were updated to match the new refresh/session flow.

**Relevant context**
- The PR's core claim is about a race. Updated tests should validate the guarantee, not just the rewritten implementation.

**What to inspect**
- Do tests check that the old refresh token is rejected after rotation?
- Is there any concurrent refresh scenario, retry scenario, or failure-path coverage?
- Do middleware tests preserve old null/error semantics expectations?

**Potential concerns**
- **Observed fact:** tests were updated for the new flow.
- **Inference:** the suite likely covers the intended happy path.
- **Risk hypothesis:** the tests may not actually prove the race is fixed; they may only prove the new sequence works once.

**Questions**
- Is there a test for two near-simultaneous refresh attempts?
- Is there a test for store failure during replacement?

**Tentative AI assessment**
- likely should-fix if concurrency/failure-path tests are absent
