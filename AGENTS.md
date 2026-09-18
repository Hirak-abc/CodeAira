# CodeAira — AGENTS.md

## 1. Project Overview

CodeAira is a LeetCode-like coding and assessment platform.

The platform includes:

- User authentication
- Registered-email OTP verification
- Coding questions
- Code editor
- Code execution and submission
- Assessment/question sessions
- Fullscreen assessment mode
- Detection of fullscreen/tab/window interruptions
- Exit verification through OTP
- Score tracking
- Completion and exit penalties
- Unexpected browser-disappearance handling
- Question resumption
- Assessment history and audit records

The implementation must follow `tasks.md` as the primary development roadmap.

---

## 2. Primary Development Rule

The agent MUST:

1. Read `tasks.md` before starting development.
2. Identify the first incomplete task.
3. Work on that task only.
4. Verify the implementation.
5. Run relevant tests/checks.
6. Mark the task complete only after verification.
7. Move to the next incomplete task.
8. Maintain a history of every meaningful change.

Do NOT skip ahead to unrelated tasks unless explicitly instructed by the user.

If a task depends on another incomplete task, resolve the dependency first or ask the user before changing the planned order.

---

## 3. Significant Decisions Require User Confirmation

Before making a significant architectural, security, data-model, product-behavior, or technology decision, STOP and ask the user.

Examples:

- Changing database technology
- Changing authentication architecture
- Changing OTP behavior
- Changing scoring/penalty rules
- Changing assessment state machine
- Changing the meaning of `+1`, `-2`, or `-3`
- Changing unexpected browser closure behavior
- Adding/removing a major feature
- Introducing a new external service/API
- Changing deployment architecture
- Changing programming language/framework
- Making destructive database/schema migrations
- Deleting an existing feature
- Changing an API contract used by multiple components
- Changing security assumptions
- Changing task order in `tasks.md`
- Making changes that could affect existing user data
- Making a major UI/UX decision not specified in `tasks.md`

Ask a concise question containing:

```text
Decision:
Options:
Recommended implementation:
Reason:
Impact:
```

Do not silently make the decision.

---

## 4. Decisions the Agent May Make Without Asking

The agent may make small implementation decisions when they do not change intended product behavior.

Examples:

- Variable names
- Function names
- Internal helper functions
- File organization within an already-defined module
- Minor CSS adjustments
- Formatting
- Refactoring duplicated code
- Adding appropriate error handling
- Writing tests
- Improving readability
- Fixing obvious bugs
- Choosing an implementation detail when `tasks.md` already specifies the behavior

The agent must still verify that the change does not conflict with `tasks.md`.

---

## 5. Do Not Invent Requirements

The agent must not invent product behavior.

If `tasks.md` specifies:

```text
Completed question → +1
```

do not change it to another score.

If a requirement is ambiguous, ask the user.

Do not infer a new business rule merely because it is technically convenient.

---

## 6. Assessment Rules Are Authoritative

These assessment behaviors must not be changed without explicit user approval.

### Completed Question

```text
Question submitted successfully
        ↓
      +1
        ↓
"Do you want to move to the next question?"
        ↓
   ┌────┴────┐
  YES        NO
   ↓          ↓
Next Q    "Do you want to quit?"
              ↓
          ┌───┴───┐
         YES      NO
          ↓        ↓
       Quit      Stay
       directly
       No OTP
       No -2
       No -3
```

### Unsubmitted Normal Exit

```text
Active question
↓
User chooses to exit/quit
↓
Exit OTP
↓
OTP verified
↓
-2
↓
Terminate session
```

### Unexpected Browser Disappearance

```text
Active/unsubmitted question
↓
Browser disappears unexpectedly
↓
Backend detects potentially interrupted session
↓
-3
↓
Session marked interrupted
↓
User returns
↓
Same question
↓
Answer reset
```

The agent must preserve these distinctions.

---

## 7. Never Trust the Frontend for Security-Critical State

Frontend events are not authoritative.

Events such as:

```text
beforeunload
unload
visibilitychange
blur
focus
fullscreenchange
```

may be used as signals, but must not be treated as definitive proof that the browser was closed.

The backend must remain authoritative for:

- Assessment state
- User identity
- OTP validity
- Question state
- Submission state
- Score
- Penalties
- Session state
- Interruption state
- Resume state

Do not implement security-critical behavior exclusively in JavaScript.

---

## 8. Fullscreen Is Not a Security Boundary

Do not assume browser fullscreen can prevent:

- Alt+Tab
- Alt+F4
- Browser termination
- OS-level termination
- Browser crashes
- Computer shutdown
- Network disconnection

The implementation should detect and handle these situations rather than claiming that the browser can completely prevent them.

If stronger kiosk-level restrictions are required, ask the user before introducing a dedicated application, managed browser, kiosk software, or other infrastructure.

---

## 9. OTP Rules

OTP must be handled securely.

The agent must:

- Never expose OTP values unnecessarily.
- Never hard-code OTPs.
- Never store plaintext OTPs when a secure alternative is available.
- Enforce expiration.
- Enforce retry/rate limits.
- Prevent OTP reuse.
- Bind OTP verification to the appropriate assessment/session context.
- Perform security-sensitive verification server-side.

Do not change OTP requirements without user confirmation.

---

## 10. Score Rules

Score changes must happen server-side.

Required rules:

```text
Successful completion → +1
Normal unsubmitted exit with verified exit OTP → -2
Unexpected active-session disappearance → -3
```

Important:

```text
Completed question + normal quit
→ +1 remains
→ no OTP
→ no -2
→ no -3
```

and:

```text
Unexpected disappearance
→ exactly one -3
```

The same interruption must never receive multiple `-3` penalties.

Use atomic/idempotent operations where necessary.

---

## 11. Unexpected Browser Closure

The agent must not depend on the frontend to tell the backend that the browser closed.

Use server-side mechanisms such as:

- Assessment session state
- Heartbeats
- `last_seen_at`
- Session timeout/staleness rules
- Persisted audit events

Temporary network failures must not automatically become a forced-close penalty if the session recovers within the configured threshold.

The backend should represent uncertain situations appropriately, for example:

```text
SESSION_UNRESOLVED
```

rather than pretending to know the exact reason for disappearance.

---

## 12. Resume Behavior

If an active question is determined to have been unexpectedly interrupted:

```text
User returns
↓
Find unresolved session
↓
Resolve interruption according to backend state
↓
Apply -3 exactly once
↓
Return to same question
↓
Reset previous answer/code
```

The system must not:

- Advance to another question
- Restore the interrupted unsaved answer
- Treat the old answer as submitted
- Apply the penalty multiple times

---

## 13. Database Changes

Before making database changes:

1. Check the existing schema.
2. Check how affected data is used.
3. Check existing migrations.
4. Check whether existing data could be affected.
5. Ask the user before destructive or significant migration.

Never:

- Drop production tables
- Delete user data
- Reset the database
- Overwrite migrations
- Remove columns containing user data

without explicit approval.

---

## 14. File Modification Rules

Before editing a file:

1. Read the relevant file.
2. Understand the existing implementation.
3. Check whether another task modifies the same area.
4. Make the smallest appropriate change.
5. Verify the result.

Do not overwrite entire files unnecessarily.

Do not delete working functionality merely to simplify implementation.

---

## 15. Required Change History

CodeAira must maintain:

```text
history.txt
```

Every meaningful edit must be recorded.

Each entry should contain:

```text
Date:
Task:
Files edited:
Changes:
Reason:
Verification:
Notes:
```

Example:

```text
Date: 2026-09-18
Task: TASK-025B — Assessment Heartbeat

Files edited:
- backend/session.py
- frontend/assessment.js
- tests/test_session.py

Changes:
- Added server-side assessment heartbeat.
- Added last_seen_at tracking.
- Added configurable stale-session threshold.

Reason:
Required for detecting potentially interrupted browser sessions
without relying on frontend browser-close events.

Verification:
- Unit tests passed.
- Heartbeat timeout test passed.
- Temporary network interruption test passed.

Notes:
No scoring behavior changed.
```

The history file must be append-only.

Do not delete previous history entries.

Do not rewrite old entries to hide previous changes.

If a mistake is corrected, add a new history entry explaining the correction.

---

## 16. History Must Be Updated Immediately

After completing a meaningful task:

1. Update `history.txt`.
2. Include every edited file.
3. Describe what changed.
4. Describe why it changed.
5. Record verification results.
6. Then update the corresponding task status in `tasks.md`.

Do not wait until the end of the project to create the history.

---

## 17. Testing Requirements

Every implemented task must be verified.

Depending on the task, verification may include:

- Unit tests
- Integration tests
- API tests
- Database tests
- Frontend tests
- Manual browser testing
- Security testing
- State-transition testing

For assessment logic, test both successful and failure paths.

At minimum verify:

```text
+1 completion
-2 normal unsubmitted exit
-3 unexpected browser disappearance
duplicate penalty prevention
resume same question
answer reset
OTP expiration
OTP reuse prevention
```

Do not mark a task complete merely because the code compiles.

---

## 18. Error Handling

Handle expected failures explicitly:

- Invalid OTP
- Expired OTP
- Too many OTP attempts
- Session not found
- Session already completed
- Duplicate submission
- Duplicate penalty request
- Network interruption
- Database failure
- Code execution failure
- Browser interruption
- Concurrent requests

Do not silently ignore errors.

Do not expose sensitive backend information to users.

---

## 19. Concurrency and Idempotency

The following operations must be safe against duplicate requests:

- Question submission
- Completion scoring
- `+1`
- Exit penalty
- `-2`
- Unexpected closure penalty
- `-3`
- Session termination
- OTP verification
- Resume

Example:

```text
Two identical completion requests
        ↓
Only one +1
```

and:

```text
Two interruption requests
        ↓
Only one -3
```

---

## 20. API Design

Before changing an existing API:

- Check all callers.
- Check frontend usage.
- Check tests.
- Check authentication requirements.
- Check whether existing clients depend on the response format.

If the API change is significant or breaking, ask the user first.

Prefer backward-compatible changes where practical.

---

## 21. Security Checklist

Before considering the assessment system complete:

- [ ] Authentication is enforced.
- [ ] OTP is server-verified.
- [ ] OTP expires.
- [ ] OTP cannot be reused.
- [ ] OTP attempts are rate-limited.
- [ ] Assessment state is server-controlled.
- [ ] Score is server-controlled.
- [ ] Penalties are server-controlled.
- [ ] Duplicate penalties are prevented.
- [ ] Duplicate completion points are prevented.
- [ ] Users cannot submit another user's question/session.
- [ ] Session IDs cannot access another user's assessment.
- [ ] Sensitive information is not exposed in frontend code.
- [ ] Browser events are not trusted as security boundaries.
- [ ] Unexpected browser disappearance can be handled without frontend cooperation.

---

## 22. Do Not Skip Verification

Do not claim a task is done until the implementation has actually been checked.

When finishing a task, report:

```text
Task:
Status:

Files changed:

Implementation:

Verification:

Remaining issues:
```

If verification cannot be performed, state that clearly and do not claim the task passed.

---

## 23. Task Completion Format

When finishing a task:

```text
TASK-XXX — COMPLETE

Implemented:
- ...

Files changed:
- ...

Verification:
- ...

History:
- Updated history.txt

Next task:
- TASK-XXX
```

Only mark a task complete when its acceptance criteria are satisfied.

---

## 24. If a Task Is Ambiguous

STOP before implementing the ambiguous portion.

Ask the user a focused question.

Example:

```text
TASK-XXX requires handling an interrupted session, but it does not specify
whether the interruption penalty should occur immediately when the heartbeat
becomes stale or only after the user returns.

Which behavior should CodeAira use?

A. Apply the penalty when the server marks the session unresolved.
B. Apply the penalty when the user returns.
```

Do not silently choose a product rule.

---

## 25. If Existing Implementation Conflicts With `tasks.md`

Do not immediately rewrite it.

First:

1. Identify the conflict.
2. Determine whether existing behavior is intentional.
3. Check whether another task explains it.
4. Ask the user if the conflict represents a significant product decision.

Do not remove existing behavior simply because a newer implementation seems cleaner.

---

## 26. Dependency Rule

Before implementing a task, check:

```text
What does this task depend on?
```

If dependencies are incomplete:

- Implement the dependency first if clearly required and already specified.
- Otherwise ask the user before changing task order.

Never mark a task complete while its required dependency is missing.

---

## 27. Do Not Change `tasks.md` Casually

`tasks.md` defines the implementation roadmap.

The agent may update:

- Completion checkboxes
- Verification notes
- Clearly completed acceptance criteria

The agent must ask the user before:

- Removing requirements
- Changing scoring rules
- Changing security behavior
- Reordering major phases
- Adding major requirements
- Removing major requirements
- Changing product behavior

---

## 28. Development Workflow

For every task:

```text
READ tasks.md
      ↓
Find first incomplete task
      ↓
Read relevant source files
      ↓
Check dependencies
      ↓
Determine whether a significant decision is required
      ↓
 ┌───────────────┐
 │ Significant?  │
 └───────┬───────┘
         │
    YES  │  NO
         │
 Ask user│
         │
         ↓
 Understand implementation
      ↓
Implement smallest correct change
      ↓
Run tests/checks
      ↓
Inspect result
      ↓
Update history.txt
      ↓
Mark task complete in tasks.md
      ↓
Move to next incomplete task
```

---

## 29. Priority Order

When requirements conflict, use this priority:

```text
1. Explicit user instructions
2. Security requirements
3. tasks.md acceptance criteria
4. Existing project architecture
5. Tests
6. Implementation convenience
```

If following a higher-priority requirement requires changing a significant existing architectural decision, ask the user before making that change when practical.

---

## 30. Definition of Done

A CodeAira task is complete only when:

- [ ] Requirements from `tasks.md` are implemented.
- [ ] Relevant existing code was inspected.
- [ ] No unrelated functionality was unnecessarily changed.
- [ ] Appropriate tests/checks were performed.
- [ ] Security implications were considered.
- [ ] Database changes were verified.
- [ ] Edge cases were considered.
- [ ] `history.txt` was updated.
- [ ] `tasks.md` was updated.
- [ ] No unresolved significant decision was silently made.

---

## 31. Final Rule

**Proceed task-by-task.**

**Do not invent requirements.**

**Do not silently make significant product or architectural decisions.**

**Ask the user before significant decisions.**

**Verify every implementation.**

**Keep `history.txt` updated with every meaningful edit and the files affected.**

**Keep `tasks.md` as the source of truth for the development roadmap.**
