## Review Brief

- **Change type:** bugfix + refactor (mixed but tightly related)
- **Scope:** 6 files across auth core, session middleware, and tests
- **Probable intent:** fix a race in refresh-token rotation while consolidating token replacement/session handling
- **Risk hotspots:** refresh-token invalidation semantics, atomicity of token replacement, middleware compatibility, adequacy of updated tests
- **Suggested review order:** token store contract → refresh flow → middleware integration → tests
- **Confidence in inferred intent:** high

## Why this order

The highest-risk behavior sits in the refresh/token lifecycle itself. Middleware and tests should be reviewed in terms of whether they preserve or correctly validate that contract.
