## Context Cards

### Context Card — Auth token rotation

- **Role in system:** owns refresh-token verification, invalidation, and replacement semantics
- **Files loaded:** `auth/refresh.ts`, `auth/token_store.ts`, `tests/auth/refresh.test.ts`
- **Why it matters:** this is the correctness and security core of the PR
- **Primary risks:** replay of old refresh token, partial update between token issue and store persistence, incompatible assumptions at call sites
- **Important assumptions:** refresh tokens should become single-use; token replacement should leave no window where both old and new tokens remain valid

### Context Card — Session middleware

- **Role in system:** maps request/session state into authenticated request context used by downstream handlers
- **Files loaded:** `server/middleware/session.ts`, `server/app.ts`, `tests/server/session-middleware.test.ts`
- **Why it matters:** middleware changes can silently alter auth behavior for otherwise unrelated routes
- **Primary risks:** changed null/error semantics, ordering regressions, inconsistent session state after token replacement
- **Important assumptions:** downstream handlers likely rely on stable behavior for missing session, invalid token, and refreshed token paths

### Context Card — Tests as contract anchors

- **Role in system:** reveal whether the PR preserves the intended behavior or only the new implementation details
- **Files loaded:** both auth and middleware tests
- **Why it matters:** updated tests may validate happy-path behavior while missing concurrency or failure-mode guarantees
- **Primary risks:** tests overfit the new implementation and fail to check the original race or rollback behavior
- **Important assumptions:** the PR claims to fix a race, so tests should say something meaningful about concurrency or single-use refresh behavior
