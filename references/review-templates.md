# Review Buddy Templates

These templates are optional output scaffolds for guided reviews, PR summaries, and comment drafting.

---

## 1. Review Brief Template

```text
## Review Brief

- Change type:
- Scope:
- Probable intent:
- Risk hotspots:
- Suggested review order:
- Confidence in inferred intent:
```

---

## 2. Context Card Template

```text
### Context Card — <module/subsystem>

- Role in system:
- Files loaded:
- Why it matters:
- Primary risks:
- Important assumptions:
```

---

## 3. Chunk Plan Template

```text
## Chunk Plan

### Chunk 1 — <name>
- Files:
- Intent:
- Why separate:
- Primary lenses:
- Potential failure modes:
- Review order:

### Chunk 2 — <name>
- Files:
- Intent:
- Why separate:
- Primary lenses:
- Potential failure modes:
- Review order:
```

---

## 4. Guided Chunk Review Template

```text
### Chunk <N> — <name>

**What changed**
- ...

**Relevant context**
- ...

**What to inspect**
- ...
- ...
- ...

**Potential concerns**
- Observed fact:
- Inference:
- Risk hypothesis:

**Questions**
- ...
- ...

**Tentative AI assessment**
- looks sound / needs clarification / likely non-blocking issue / likely blocking issue
```

---

## 5. Final Summary Template

```text
## Final Review Summary

**Recommended verdict**
- Approve / Comment / Request changes / Needs more context

### Blocking issues
- ...

### Should fix
- ...

### Discuss
- ...

### Nit
- ...

### Positive notes
- ...

### Confidence / remaining uncertainty
- ...
```

---

## 6. Inline Comment Template

```text
<concern>

Why it matters:
<impact or risk>

Suggestion / question:
<proposed change or clarification request>
```

Example:

```text
This changes the token invalidation path, but I can't tell whether the old refresh token can still be replayed if persistence succeeds after token issuance.

Why it matters:
This is a security-sensitive state transition, so a partial update could leave the old token valid longer than intended.

Suggestion / question:
Should this update be atomic or fail closed if persistence does not complete before the new token is returned?
```

---

## 7. Author Question Template

```text
Question:
<clarification request>

Why I'm asking:
<what ambiguity affects the review judgment>
```

Example:

```text
Question:
Is the compatibility change here intentional? The old helper returned `null`, while this path now throws.

Why I'm asking:
If callers still rely on the old contract, this is a behavioral break rather than a pure refactor.
```

---

## 8. Summary Comment Template

```text
## Review Summary

**Verdict:** Approve / Comment / Request changes

### Blocking
- ...

### Important concerns
- ...

### Questions
- ...

### Positive notes
- ...
```

---

## 9. AI-Generated Change Review Prompts

### Prompt: High-risk chunk focus

```text
Focus on the highest-risk chunk first. Summarize what changed, what context matters, what invariants should still hold, and what concrete failure modes the reviewer should inspect.
```

### Prompt: Context bootstrap

```text
Before reviewing the diff, load the smallest surrounding context needed to understand intent: related tests, callers/callees, interface definitions, and nearby unchanged code. Summarize the subsystem and the key assumptions.
```

### Prompt: Agent-written refactor skepticism

```text
Treat this as agent-generated code that may look polished while hiding shallow understanding. Check for semantic drift, partial migration, missing edge-case tests, and preserved names with changed meaning.
```
