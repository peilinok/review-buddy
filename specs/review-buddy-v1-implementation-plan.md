# Review Buddy V1 Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.

**Goal:** Build Review Buddy V1 as an agent-invocable local web review surface that prepares a review session from repo changes or commit ranges, launches a browser UI, and supports chunked diff review, discussion threads, AI replies, findings, and export.

**Architecture:** The system is split into three layers: an **agent invocation layer** that recognizes Review Buddy requests and resolves the review target, a **review session preparation layer** that snapshots diffs and materializes core review objects, and a **local web app layer** that renders the session and drives interactive review. All storage, APIs, and UI behavior are organized around the core object relationships defined in `specs/review-buddy-web-v1.md`.

**Tech Stack:** Keep the repo centered on `review-buddy`; implement a local HTTP server plus browser UI with structured JSON persistence. Prefer a lightweight TypeScript web stack if starting fresh, but keep the plan implementation-agnostic enough that the first step is creating the app scaffold under this repo rather than overcommitting to a framework before the skeleton exists.

---

## 0. Preconditions and design anchors

Before implementation starts, the implementer must read:

- `./specs/review-buddy-web-v1.md`
- `./SKILL.md`
- `./README.md`
- `./references/review-templates.md`

The implementation must preserve these invariants:

1. **Review Buddy stays one project and one skill.**
2. **The primary entrypoint is the agent, not manual UI-first startup.**
3. **All behavior maps back to core objects:** `ReviewSession`, `DiffFile`, `DiffHunk`, `ReviewChunk`, `CommentThread`, `Comment`, `AIMessage`, `SummaryFinding`.
4. **The right pane is structured guidance + threads, not general chat.**
5. **V1 must work locally without requiring hosted infrastructure.**

---

## 1. Proposed repo additions

Add implementation-oriented directories under the existing `review-buddy` project.

```text
review-buddy/
├── SKILL.md
├── README.md
├── specs/
│   └── review-buddy-web-v1.md
├── references/
├── examples/
├── docs/
│   └── implementation/
│       └── review-buddy-v1-architecture.md
├── app/
│   ├── server/
│   ├── client/
│   └── shared/
├── fixtures/
│   └── diffs/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
└── scripts/
```

### Directory intent

- `app/server/` — local session prep, persistence, launch APIs
- `app/client/` — local web UI
- `app/shared/` — core object schemas, types, protocol contracts
- `fixtures/diffs/` — saved diff fixtures for deterministic tests
- `tests/unit/` — object model, parsers, reducers
- `tests/integration/` — session prep, persistence, exports, launch flow
- `tests/e2e/` — browser-driven end-to-end review flow
- `docs/implementation/` — architecture notes beyond the product spec

---

## 2. System architecture overview

Implement the system in three layers.

### Layer A — Agent invocation layer

Responsible for:

- detecting Review Buddy intent from natural language
- resolving the review target
- selecting the correct diff source mode
- invoking session preparation
- launching the local UI and returning the session entrypoint

### Layer B — Review session preparation layer

Responsible for:

- reading git or pasted diff input
- normalizing diff files and hunks
- materializing core objects
- generating review brief and chunk candidates
- persisting a resumable `ReviewSession`

### Layer C — Local web app layer

Responsible for:

- rendering diff, chunks, and threads
- collecting reviewer actions
- sending structured requests for AI follow-up
- managing thread lifecycle and finding lifecycle
- exporting review artifacts

### Cross-cutting concerns

- JSON schemas / types for core objects
- local persistence and migrations
- test fixtures and deterministic e2e flows
- browser launch / session URL handoff

---

## 3. Core object implementation strategy

Implement the core object model first. Do not build the UI first and invent state ad hoc.

### Objects to define in `app/shared/`

- `ReviewSession`
- `DiffFile`
- `DiffHunk`
- `ReviewChunk`
- `CommentThread`
- `Comment`
- `CommentAnchor`
- `AIMessage`
- `SummaryFinding`

### Requirements

- create canonical schema/type definitions
- validate all read/write payloads against those definitions
- centralize IDs, status enums, and severity enums
- define serialization shape once and reuse it everywhere

### Suggested files

```text
app/shared/
├── types.ts
├── schemas.ts
├── ids.ts
├── enums.ts
└── protocol.ts
```

---

## 4. Implementation phases

Build V1 in this order:

1. project scaffold + core schemas
2. diff ingestion and snapshot persistence
3. session preparation pipeline
4. local server + session APIs
5. minimal browser UI shell
6. chunk navigation + diff rendering
7. discussion thread system
8. AI response plumbing
9. findings and exports
10. full end-to-end launch from agent request

Do not start with styling or advanced UX polish.

---

# Phase 1 — Core schemas and project scaffold

### Task 1: Create implementation scaffolding

**Objective:** Add the baseline app, tests, fixtures, and docs directories so implementation has a stable home.

**Files:**
- Create: `app/server/`
- Create: `app/client/`
- Create: `app/shared/`
- Create: `tests/unit/`
- Create: `tests/integration/`
- Create: `tests/e2e/`
- Create: `fixtures/diffs/`
- Create: `docs/implementation/review-buddy-v1-architecture.md`

**Step 1: Create directories**

Create the directory tree listed in Section 1.

**Step 2: Add README stubs**

Create a short `README.md` in `app/` or `docs/implementation/` describing the three-layer architecture.

**Step 3: Verify structure**

Run a directory listing command that confirms the new structure exists.

**Step 4: Commit**

```bash
git add app tests fixtures docs
git commit -m "chore: scaffold review buddy app structure"
```

---

### Task 2: Define core enums and IDs

**Objective:** Establish one authoritative source for statuses, severities, and ID helpers.

**Files:**
- Create: `app/shared/enums.ts`
- Create: `app/shared/ids.ts`
- Test: `tests/unit/shared/test_enums_and_ids.*`

**Step 1: Write failing test**

Test for:

- valid thread statuses: `open`, `resolved`, `dismissed`
- valid finding statuses: `candidate`, `accepted`, `rejected`
- valid severity values
- generated IDs with prefixes like `session-`, `thread-`, `finding-`

**Step 2: Run test to verify failure**

Run the unit test command for this file. Expected: missing module or missing exports.

**Step 3: Implement enums and ID helpers**

Add:

- thread status constants
- finding status constants
- severity constants
- helper functions like `newSessionId()`, `newThreadId()`, `newFindingId()`

**Step 4: Run test to verify pass**

Expected: all enum / ID tests pass.

**Step 5: Commit**

```bash
git add app/shared/enums.ts app/shared/ids.ts tests/unit/shared
git commit -m "feat: add core enums and id helpers"
```

---

### Task 3: Define core object schemas and shared types

**Objective:** Make the object model executable and validated.

**Files:**
- Create: `app/shared/types.ts`
- Create: `app/shared/schemas.ts`
- Test: `tests/unit/shared/test_core_schemas.*`

**Step 1: Write failing test**

Test round-trip validation for:

- `ReviewSession`
- `ReviewChunk`
- `CommentThread`
- `Comment`
- `SummaryFinding`

**Step 2: Run test to verify failure**

Expected: schema definitions missing.

**Step 3: Implement shared types and schemas**

Add one canonical type/schema per core object from the spec.

**Step 4: Run test to verify pass**

Expected: valid fixtures pass validation; malformed payloads fail.

**Step 5: Commit**

```bash
git add app/shared/types.ts app/shared/schemas.ts tests/unit/shared
git commit -m "feat: define review buddy core object schemas"
```

---

# Phase 2 — Diff ingestion and snapshot persistence

### Task 4: Add diff fixture set

**Objective:** Create deterministic diff inputs for unit and e2e tests.

**Files:**
- Create: `fixtures/diffs/working-tree.patch`
- Create: `fixtures/diffs/branch-diff.patch`
- Create: `fixtures/diffs/commit-range.patch`
- Create: `fixtures/diffs/mixed-auth-session.patch`

**Step 1: Build representative fixtures**

Include examples with:

- multi-file diffs
- several hunks per file
- renamed or modified files if practical
- mixed code + tests

**Step 2: Verify fixture readability**

Read them back and ensure they are valid unified diff text.

**Step 3: Commit**

```bash
git add fixtures/diffs
git commit -m "test: add review buddy diff fixtures"
```

---

### Task 5: Implement diff parser to `DiffFile` and `DiffHunk`

**Objective:** Convert unified diffs into normalized core objects.

**Files:**
- Create: `app/server/diff_parser.ts`
- Test: `tests/unit/server/test_diff_parser.*`

**Step 1: Write failing test**

Use fixtures to assert:

- file count
- hunk count
- paths preserved
- hunk headers preserved
- line ranges parsed correctly

**Step 2: Run test to verify failure**

Expected: parser missing.

**Step 3: Implement parser**

Return normalized `DiffFile` and `DiffHunk` payloads compatible with shared schemas.

**Step 4: Run test to verify pass**

Expected: parser fixtures all pass.

**Step 5: Commit**

```bash
git add app/server/diff_parser.ts tests/unit/server
git commit -m "feat: parse unified diffs into review buddy objects"
```

---

### Task 6: Implement diff source adapters

**Objective:** Support all V1 diff input modes.

**Files:**
- Create: `app/server/diff_sources.ts`
- Test: `tests/integration/server/test_diff_sources.*`

**Step 1: Write failing integration tests**

Test adapters for:

- working tree vs HEAD
- staged diff
- branch diff
- commit range
- pasted unified diff

**Step 2: Run tests to verify failure**

Expected: source adapter module missing.

**Step 3: Implement source adapters**

Each adapter should return:

- raw unified diff
- source metadata (`source_type`, `source_ref`)

**Step 4: Run tests to verify pass**

Expected: all adapters return valid diff text or fixture-backed values.

**Step 5: Commit**

```bash
git add app/server/diff_sources.ts tests/integration/server
git commit -m "feat: add review buddy diff source adapters"
```

---

### Task 7: Implement local session persistence

**Objective:** Persist `ReviewSession` and associated objects locally so sessions are resumable.

**Files:**
- Create: `app/server/session_store.ts`
- Create: `app/server/storage_layout.md` or `docs/implementation/storage.md`
- Test: `tests/integration/server/test_session_store.*`

**Step 1: Write failing integration test**

Test:

- create session
- persist diff files/hunks
- reload session
- preserve threads/findings round-trip

**Step 2: Run test to verify failure**

Expected: storage module missing.

**Step 3: Implement storage**

Prefer a simple local JSON or SQLite-backed store, but keep one canonical persistence API.

The API should support:

- `createSession(...)`
- `getSession(sessionId)`
- `saveThreads(...)`
- `saveFindings(...)`
- `listSessions()`

**Step 4: Run test to verify pass**

Expected: persisted sessions reload cleanly.

**Step 5: Commit**

```bash
git add app/server/session_store.ts docs/implementation tests/integration/server
git commit -m "feat: add local review session persistence"
```

---

# Phase 3 — Session preparation layer

### Task 8: Implement review session preparation pipeline

**Objective:** Build the orchestration path from diff source to fully materialized `ReviewSession`.

**Files:**
- Create: `app/server/session_prepare.ts`
- Test: `tests/integration/server/test_session_prepare.*`

**Step 1: Write failing integration test**

Test that a preparation request:

- resolves source
- parses diff
- creates session
- stores files/hunks
- returns session metadata

**Step 2: Run test to verify failure**

Expected: preparation pipeline missing.

**Step 3: Implement pipeline**

Expose one high-level function like:

- `prepareReviewSession(request): PreparedSessionResult`

**Step 4: Run test to verify pass**

Expected: prepared session contains normalized objects and a stable session ID.

**Step 5: Commit**

```bash
git add app/server/session_prepare.ts tests/integration/server
git commit -m "feat: implement review session preparation pipeline"
```

---

### Task 9: Implement review brief generation contract

**Objective:** Generate and persist `review_brief` for a session.

**Files:**
- Create: `app/server/review_brief.ts`
- Test: `tests/integration/server/test_review_brief.*`

**Step 1: Write failing test**

Test that a prepared session can produce a structured brief with:

- change type
- scope
- probable intent
- risk hotspots
- review order suggestion

**Step 2: Run test to verify failure**

Expected: missing generator.

**Step 3: Implement brief generation interface**

Stub with deterministic test implementation if needed, but keep the public contract aligned with the spec.

**Step 4: Run test to verify pass**

Expected: brief object saved on session.

**Step 5: Commit**

```bash
git add app/server/review_brief.ts tests/integration/server
git commit -m "feat: add review brief generation contract"
```

---

### Task 10: Implement best-effort chunk generation

**Objective:** Produce `ReviewChunk` candidates from parsed hunks.

**Files:**
- Create: `app/server/chunk_generator.ts`
- Test: `tests/unit/server/test_chunk_generator.*`

**Step 1: Write failing test**

Use the mixed fixture to assert:

- chunks group related hunks
- chunk order exists
- fallback to file/hunk still works if chunking is weak

**Step 2: Run test to verify failure**

Expected: chunk generator missing.

**Step 3: Implement generator**

Keep V1 simple:

- start with file-aware / path-aware grouping
- allow future AI-assisted grouping
- never block session creation if chunking confidence is low

**Step 4: Run test to verify pass**

Expected: chunk candidates created deterministically enough for fixtures.

**Step 5: Commit**

```bash
git add app/server/chunk_generator.ts tests/unit/server
git commit -m "feat: add best effort review chunk generator"
```

---

# Phase 4 — Local server and launch APIs

### Task 11: Build local server shell

**Objective:** Start a local server that can serve session JSON and a browser UI.

**Files:**
- Create: `app/server/main.*`
- Create: `app/server/routes.*`
- Test: `tests/integration/server/test_server_health.*`

**Step 1: Write failing test**

Test:

- server starts
- health endpoint responds
- static UI or placeholder page is served

**Step 2: Run test to verify failure**

Expected: server missing.

**Step 3: Implement server shell**

Add at minimum:

- `/health`
- `/sessions/:id`
- `/sessions/:id/export/*`

**Step 4: Run test to verify pass**

Expected: all server health checks pass.

**Step 5: Commit**

```bash
git add app/server/main.* app/server/routes.* tests/integration/server
git commit -m "feat: add review buddy local server shell"
```

---

### Task 12: Implement browser launch helper

**Objective:** Open the prepared session in the default browser.

**Files:**
- Create: `app/server/launch_browser.*`
- Test: `tests/unit/server/test_launch_browser.*`

**Step 1: Write failing test**

Test that launch helper builds a session URL and invokes the OS/browser opening path.

**Step 2: Run test to verify failure**

Expected: helper missing.

**Step 3: Implement launch helper**

It should:

- accept `session_id`
- build local URL
- open browser
- return URL for agent display/logging

**Step 4: Run test to verify pass**

Expected: launch helper returns expected URL and uses mocked opener.

**Step 5: Commit**

```bash
git add app/server/launch_browser.* tests/unit/server
git commit -m "feat: add browser launch helper for review sessions"
```

---

### Task 13: Implement agent-facing launch entrypoint

**Objective:** Expose one entrypoint the agent can call to go from request -> session -> browser launch.

**Files:**
- Create: `app/server/launch_review_buddy.*`
- Test: `tests/integration/server/test_launch_review_buddy.*`

**Step 1: Write failing integration test**

Test that a launch request:

- resolves diff source
- prepares session
- starts server if needed
- opens browser URL
- returns session and URL metadata

**Step 2: Run test to verify failure**

Expected: launch entrypoint missing.

**Step 3: Implement launch entrypoint**

Suggested return shape:

```json
{
  "session_id": "session-123",
  "url": "http://127.0.0.1:.../sessions/session-123",
  "source_type": "branch_diff"
}
```

**Step 4: Run test to verify pass**

Expected: launch path works end-to-end with mocked browser opener.

**Step 5: Commit**

```bash
git add app/server/launch_review_buddy.* tests/integration/server

git commit -m "feat: add review buddy launch entrypoint"
```

---

# Phase 5 — Local web app shell

### Task 14: Create client app shell

**Objective:** Build the base UI layout matching the spec.

**Files:**
- Create: `app/client/src/App.*`
- Create: `app/client/src/layout/*`
- Test: `tests/e2e/test_app_shell.*`

**Step 1: Write failing e2e test**

Test that the session page renders:

- left nav
- center diff area
- right panel

**Step 2: Run test to verify failure**

Expected: client shell missing.

**Step 3: Implement shell**

Render placeholder panes wired to session data.

**Step 4: Run test to verify pass**

Expected: all three panes visible.

**Step 5: Commit**

```bash
git add app/client tests/e2e
git commit -m "feat: create review buddy app shell"
```

---

### Task 15: Render session and chunk navigation

**Objective:** Populate the left pane from `ReviewSession`, `ReviewChunk`, and file objects.

**Files:**
- Create: `app/client/src/features/navigation/*`
- Test: `tests/e2e/test_navigation.*`

**Step 1: Write failing e2e test**

Test:

- session title shown
- chunk list shown
- file list shown
- clicking chunk updates active scope

**Step 2: Run test to verify failure**

Expected: nav components missing.

**Step 3: Implement navigation**

Show chunk counts and thread indicators where practical.

**Step 4: Run test to verify pass**

Expected: navigation updates center/right panes.

**Step 5: Commit**

```bash
git add app/client/src/features/navigation tests/e2e
git commit -m "feat: add session and chunk navigation"
```

---

### Task 16: Render unified diff view

**Objective:** Display `DiffFile` and `DiffHunk` data in the center pane.

**Files:**
- Create: `app/client/src/features/diff-view/*`
- Test: `tests/e2e/test_diff_view.*`

**Step 1: Write failing e2e test**

Test:

- file diff appears
- hunk headers appear
- additions/deletions styled
- line numbers visible

**Step 2: Run test to verify failure**

Expected: diff view missing.

**Step 3: Implement unified diff renderer**

Keep V1 to unified view only.

**Step 4: Run test to verify pass**

Expected: diff fixture renders correctly.

**Step 5: Commit**

```bash
git add app/client/src/features/diff-view tests/e2e
git commit -m "feat: render unified diff view"
```

---

# Phase 6 — Discussion thread system

### Task 17: Add line-range thread creation

**Objective:** Allow reviewers to create inline threads from diff selections.

**Files:**
- Create: `app/client/src/features/threads/create-thread/*`
- Test: `tests/e2e/test_inline_thread_creation.*`

**Step 1: Write failing e2e test**

Test:

- select diff lines
- create thread
- thread appears in right pane
- anchor marker appears in diff

**Step 2: Run test to verify failure**

Expected: no thread creation flow.

**Step 3: Implement thread creation UI**

Persist `CommentThread`, `Comment`, and `CommentAnchor`.

**Step 4: Run test to verify pass**

Expected: thread creation works end-to-end.

**Step 5: Commit**

```bash
git add app/client/src/features/threads tests/e2e
git commit -m "feat: add inline discussion thread creation"
```

---

### Task 18: Add chunk and global thread creation

**Objective:** Support thread creation at chunk and review scope.

**Files:**
- Modify: `app/client/src/features/threads/*`
- Test: `tests/e2e/test_chunk_and_global_threads.*`

**Step 1: Write failing e2e test**

Test:

- create chunk thread
- create global thread
- each appears with correct scope label

**Step 2: Run test to verify failure**

Expected: only inline path exists.

**Step 3: Implement scope-aware thread creation**

Distinguish `inline`, `chunk`, and `global` scopes in UI and persistence.

**Step 4: Run test to verify pass**

Expected: all three thread scopes work.

**Step 5: Commit**

```bash
git add app/client/src/features/threads tests/e2e
git commit -m "feat: support chunk and global discussion threads"
```

---

### Task 19: Add thread lifecycle controls

**Objective:** Support resolve, dismiss, and reopen thread flows.

**Files:**
- Modify: `app/client/src/features/threads/*`
- Modify: `app/server/routes.*`
- Test: `tests/e2e/test_thread_lifecycle.*`

**Step 1: Write failing e2e test**

Test:

- resolve thread
- reopen thread
- dismiss thread
- status badge updates correctly

**Step 2: Run test to verify failure**

Expected: lifecycle actions missing.

**Step 3: Implement lifecycle actions**

Persist thread status changes and reflect them in the UI.

**Step 4: Run test to verify pass**

Expected: status transitions succeed.

**Step 5: Commit**

```bash
git add app/client app/server tests/e2e
git commit -m "feat: add discussion thread lifecycle controls"
```

---

# Phase 7 — AI guidance and follow-up plumbing

### Task 20: Render review brief and chunk guide

**Objective:** Show AI artifacts outside threads using `AIMessage` objects.

**Files:**
- Create: `app/client/src/features/ai-guide/*`
- Test: `tests/e2e/test_ai_guide.*`

**Step 1: Write failing e2e test**

Test:

- review brief appears
- chunk guide appears when chunk selected
- right pane does not look like open-ended chat

**Step 2: Run test to verify failure**

Expected: guide panel missing.

**Step 3: Implement AI Guide panel**

Bind review brief and chunk explainer data to the right pane.

**Step 4: Run test to verify pass**

Expected: guide content updates by scope.

**Step 5: Commit**

```bash
git add app/client/src/features/ai-guide tests/e2e
git commit -m "feat: render ai review brief and chunk guide"
```

---

### Task 21: Send reviewer comments and store AI replies

**Objective:** Complete the structured discussion loop.

**Files:**
- Create: `app/server/ai_reply.ts`
- Modify: `app/server/routes.*`
- Modify: `app/client/src/features/threads/*`
- Test: `tests/integration/server/test_ai_reply.*`
- Test: `tests/e2e/test_thread_ai_reply.*`

**Step 1: Write failing tests**

Test:

- reviewer posts comment
- server sends structured request
- AI reply returns
- reply stored as `Comment(author="ai")`
- reply appears in thread

**Step 2: Run tests to verify failure**

Expected: AI reply plumbing missing.

**Step 3: Implement structured reply path**

Return:

- AI reply text
- severity suggestion
- suggested draft
- follow-up hints

**Step 4: Run tests to verify pass**

Expected: AI reply appears inside the correct thread.

**Step 5: Commit**

```bash
git add app/server app/client tests/integration tests/e2e
git commit -m "feat: add ai replies inside discussion threads"
```

---

# Phase 8 — Findings and exports

### Task 22: Implement finding creation and promotion

**Objective:** Convert discussion into structured review findings.

**Files:**
- Create: `app/client/src/features/findings/*`
- Modify: `app/server/routes.*`
- Test: `tests/e2e/test_findings.*`

**Step 1: Write failing e2e test**

Test:

- promote finding from thread
- edit finding
- accept or reject finding
- summary updates accordingly

**Step 2: Run test to verify failure**

Expected: finding UI missing.

**Step 3: Implement finding workflow**

Support:

- candidate
- accepted
- rejected

Keep source references to thread/chunk/hunk.

**Step 4: Run test to verify pass**

Expected: findings flow works end-to-end.

**Step 5: Commit**

```bash
git add app/client app/server tests/e2e
git commit -m "feat: add finding promotion and acceptance workflow"
```

---

### Task 23: Implement exports

**Objective:** Produce markdown, PR-ready markdown, and JSON outputs.

**Files:**
- Create: `app/server/export_review.ts`
- Test: `tests/integration/server/test_exports.*`

**Step 1: Write failing integration tests**

Test export outputs for:

- session JSON
- markdown review summary
- PR-ready markdown draft

**Step 2: Run tests to verify failure**

Expected: export module missing.

**Step 3: Implement export writers**

Include:

- summary buckets
- selected findings
- thread-derived comment drafts where appropriate

**Step 4: Run tests to verify pass**

Expected: export artifacts generated with stable content.

**Step 5: Commit**

```bash
git add app/server/export_review.ts tests/integration/server
git commit -m "feat: add review export formats"
```

---

# Phase 9 — Agent-first end-to-end flow

### Task 24: Implement Review Buddy request resolver

**Objective:** Detect and normalize agent-side Review Buddy requests.

**Files:**
- Create: `app/server/request_resolver.*`
- Test: `tests/unit/server/test_request_resolver.*`

**Step 1: Write failing tests**

Test request parsing for phrases like:

- review current project changes
- review branch vs main
- review these commits
- review pasted diff

**Step 2: Run tests to verify failure**

Expected: resolver missing.

**Step 3: Implement resolver**

Normalize into a structured launch request object.

**Step 4: Run tests to verify pass**

Expected: resolver outputs valid target metadata.

**Step 5: Commit**

```bash
git add app/server/request_resolver.* tests/unit/server
git commit -m "feat: resolve agent review buddy requests"
```

---

### Task 25: Add end-to-end launch test from request to UI

**Objective:** Prove the entire V1 promise works.

**Files:**
- Test: `tests/e2e/test_agent_launch_to_review_ui.*`

**Step 1: Write failing e2e/integration hybrid test**

Test:

- structured launch request created
- session prepared
- browser URL returned
- session page loads
- diff visible
- chunk navigation visible

**Step 2: Run test to verify failure**

Expected: one or more links in the chain incomplete.

**Step 3: Fill missing glue**

Add any missing route wiring, payload shape fixes, or UI hydration needed to make the flow real.

**Step 4: Run test to verify pass**

Expected: end-to-end launch succeeds.

**Step 5: Commit**

```bash
git add tests/e2e app/server app/client
git commit -m "feat: complete agent to review ui launch flow"
```

---

# Phase 10 — Documentation and polish

### Task 26: Update README for the new product shape

**Objective:** Document Review Buddy as skill + local web app workflow.

**Files:**
- Modify: `README.md`
- Test: manual doc review

**Step 1: Update README sections**

Add:

- what Review Buddy now is
- agent-first invocation examples
- local review flow
- export artifacts

**Step 2: Verify wording matches spec**

Check consistency with `specs/review-buddy-web-v1.md`.

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: document review buddy local web workflow"
```

---

### Task 27: Update `SKILL.md` to reflect UI-launch behavior

**Objective:** Keep the skill definition aligned with the product implementation.

**Files:**
- Modify: `SKILL.md`
- Test: readback review

**Step 1: Update skill overview and workflow**

Document that Review Buddy can:

- analyze review target
- prepare session
- launch interactive local review surface

**Step 2: Verify skill remains coherent**

Ensure the skill still explains the original review methodology, but now includes the UI handoff behavior.

**Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: align review buddy skill with local review ui"
```

---

## 5. Testing strategy

### Unit tests

Cover:

- enums and IDs
- schemas
- diff parser
- chunk generator
- request resolver
- browser launch helper

### Integration tests

Cover:

- diff source adapters
- session preparation
- persistence
- brief generation contract
- AI reply plumbing
- exports
- launch entrypoint

### E2E tests

Cover:

- app shell
- navigation
- diff rendering
- inline/chunk/global thread creation
- thread lifecycle
- AI replies in thread
- findings workflow
- agent-launch-to-UI flow

### Minimum verification before calling V1 done

- one real branch diff launches into browser UI
- one commit-range review launches into browser UI
- reviewer can create a thread and get an AI reply
- reviewer can promote one finding and export results

---

## 6. Suggested implementation order if time is tight

If implementation must be staged aggressively, prioritize this vertical slice first:

1. core schemas
2. diff parser
3. session store
4. session preparation
5. local server
6. browser launch
7. app shell
8. diff view
9. inline thread creation
10. AI reply in thread

Only then add:

- chunk improvements
- finding workflow
- PR-ready export polish

---

## 7. Deliverables checklist

By the end of this plan, the repo should contain:

- implementation scaffold under `app/`
- shared core object schemas
- diff source and parser pipeline
- local persistence layer
- launchable local server
- browser UI with three-pane layout
- discussion thread system
- AI guidance and thread reply plumbing
- findings workflow
- markdown / JSON / PR-ready exports
- updated `README.md`
- updated `SKILL.md`
- automated tests across unit, integration, and e2e layers

---

## 8. Final verification checklist

Before declaring V1 complete, confirm:

- [ ] Review Buddy is still one skill/project identity
- [ ] The primary launch path begins from the agent
- [ ] All implemented state maps to the core object model
- [ ] The web app is structured-inspector-first, not chat-first
- [ ] All V1 diff input modes work
- [ ] Thread lifecycle works
- [ ] Finding lifecycle works
- [ ] Exports work
- [ ] End-to-end launch from agent request to review page is real

---

## 9. Execution handoff

Plan complete and saved. Ready to execute using subagent-driven-development — dispatch a fresh subagent per task with spec-compliance review and then code-quality review after each task.