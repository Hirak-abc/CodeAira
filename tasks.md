# CodeAira — TASKS.md

# TASKS.md

# CodeAira — Implementation Tasks

This file contains the complete implementation plan for the coding/assessment platform, including authentication, OTP verification, fullscreen control, exit handling, scoring, code execution, security, event logging, testing, and production hardening.

---

## Project Name

**CodeAira**

CodeAira is a LeetCode-like coding and assessment platform with controlled fullscreen assessment sessions, email OTP verification, coding-question execution/submission, completion scoring, and OTP-protected unsubmitted exits.

# Phase 1 — Project Foundation

## TASK-001 — Initialize Repository

- [ ] Create frontend project.
- [ ] Create backend project.
- [ ] Configure environment variables.
- [ ] Configure Git.
- [ ] Add `.gitignore`.
- [ ] Add `.env.example`.
- [ ] Add README.
- [ ] Establish frontend/backend directory structure.

Suggested structure:

```text
project/
├── frontend/
├── backend/
├── tests/
├── docs/
├── AGENTS.md
├── TASKS.md
├── README.md
└── .env.example
```

---

## TASK-002 — Configure Database

- [ ] Select and configure the database.
- [ ] Create database connection layer.
- [ ] Create Users model.
- [ ] Create Questions model.
- [ ] Create AssessmentSession model.
- [ ] Create Submission model.
- [ ] Create OTPRecord model.
- [ ] Create AssessmentEvent model.
- [ ] Add indexes for frequently queried fields.
- [ ] Configure database migrations/schema management if applicable.

### Users

```text
id
name
email
password_hash / auth_provider_id
overall_score
created_at
updated_at
```

### Questions

```text
id
title
description
difficulty
starter_code
constraints
examples
test_cases
supported_languages
created_at
updated_at
```

### Assessment Sessions

```text
id
user_id
question_id
status
started_at
submitted_at
ended_at
otp_verified
exit_reason
score_delta
last_seen_at
heartbeat_status
interruption_detected_at
interruption_penalty_applied
created_at
updated_at
```

### Submissions

```text
id
user_id
question_id
assessment_session_id
language
code
status
execution_result
created_at
```

### OTP Records

```text
id
user_id
assessment_session_id
purpose
otp_hash
expires_at
attempt_count
verified_at
created_at
```

OTP purposes:

```text
START_ASSESSMENT
EXIT_ASSESSMENT
```

### Assessment Events

```text
id
user_id
assessment_session_id
event_type
timestamp
metadata
```

---

# Phase 2 — Authentication

## TASK-003 — Implement User Authentication

- [ ] Implement registration/login.
- [ ] Implement session/token handling.
- [ ] Store password hashes if password authentication is used.
- [ ] Store verified email address.
- [ ] Implement logout.
- [ ] Implement authentication middleware.
- [ ] Implement authorization middleware.
- [ ] Prevent unauthenticated assessment access.

Acceptance criteria:

```text
Unauthenticated user → cannot start assessment.
Authenticated user → can access question dashboard.
```

---

## TASK-004 — User Profile

- [ ] Create profile endpoint.
- [ ] Display user's registered email.
- [ ] Display overall score.
- [ ] Display completed questions.
- [ ] Display assessment history where appropriate.

---

# Phase 3 — Question System

## TASK-005 — Question Model

Implement:

```text
Question
├── id
├── title
├── description
├── difficulty
├── constraints
├── examples
├── starter_code
├── supported_languages
└── test_cases
```

- [ ] Create database model.
- [ ] Create question CRUD APIs.
- [ ] Create question listing endpoint.
- [ ] Create question detail endpoint.
- [ ] Add validation for question data.

---

## TASK-006 — Question Dashboard

- [ ] Create question list.
- [ ] Add search.
- [ ] Add difficulty filters.
- [ ] Add question status.
- [ ] Add user's score.
- [ ] Add Start button.
- [ ] Add completed-question indicator.
- [ ] Prevent starting invalid/nonexistent questions.

---

# Phase 4 — Assessment Session

## TASK-007 — Create Assessment Session

Endpoint:

```text
POST /api/assessment/start
```

Input:

```json
{
  "question_id": "..."
}
```

Backend:

- [ ] Verify authenticated user.
- [ ] Verify question exists.
- [ ] Create unique assessment session.
- [ ] Set initial state.
- [ ] Generate secure start OTP.
- [ ] Hash OTP.
- [ ] Set OTP expiry.
- [ ] Store OTP record.
- [ ] Send OTP email.
- [ ] Return session ID and non-sensitive assessment information.
- [ ] Never return the OTP.

Recommended state:

```text
QUESTION_SELECTED
→ OTP_SENT
```

---

## TASK-008 — Assessment Session Ownership

Every assessment API must verify:

```text
authenticated?
session exists?
session belongs to current user?
question belongs to session?
operation is valid for current state?
```

- [ ] Implement reusable authorization helper/middleware.
- [ ] Add tests for unauthorized session access.

---

# Phase 5 — Start OTP

## TASK-009 — OTP Generation Service

Create:

```text
OTPService
├── generateOTP()
├── hashOTP()
├── verifyOTP()
├── isExpired()
└── invalidateOTP()
```

Recommended configuration:

```text
OTP_LENGTH = 6
OTP_EXPIRY = 5 minutes
MAX_OTP_ATTEMPTS = 5
```

- [ ] Use cryptographically secure random generation.
- [ ] Never hardcode OTPs.
- [ ] Never expose OTP through frontend APIs.
- [ ] Never log plaintext OTP in production.
- [ ] Store hashed OTP.

---

## TASK-010 — Email Service

Create:

```text
EmailService
├── sendStartOTP()
├── sendExitOTP()
└── sendEmail()
```

- [ ] Configure email provider.
- [ ] Configure SMTP/API credentials through environment variables.
- [ ] Create start-assessment OTP email template.
- [ ] Create exit-assessment OTP email template.
- [ ] Handle email delivery failures.
- [ ] Add rate limiting to OTP sending.
- [ ] Do not expose email credentials to frontend.

Start email should clearly identify:

```text
Purpose: Start Assessment
```

Exit email should clearly identify:

```text
Purpose: Exit Assessment
```

---

## TASK-011 — Start OTP Verification

Endpoint:

```text
POST /api/assessment/verify-start-otp
```

- [ ] Validate authenticated user.
- [ ] Validate assessment session.
- [ ] Validate OTP purpose.
- [ ] Validate OTP.
- [ ] Check expiry.
- [ ] Check attempt limit.
- [ ] Invalidate used OTP.
- [ ] Mark OTP as verified.
- [ ] Change assessment state to `OTP_VERIFIED`.
- [ ] Log OTP verification event.
- [ ] Return permission to enter assessment.

Acceptance:

```text
Valid OTP
→ OTP_VERIFIED
→ Fullscreen can be requested
→ Assessment can start
```

---

# Phase 6 — Fullscreen Assessment

## TASK-012 — Fullscreen Manager

Create:

```text
useFullscreen()
```

Implement:

- [ ] Enter fullscreen.
- [ ] Exit fullscreen.
- [ ] Detect fullscreen changes.
- [ ] Detect unsupported browsers.
- [ ] Maintain frontend fullscreen state.
- [ ] Synchronize relevant events with backend.

Browser API:

```js
document.documentElement.requestFullscreen()
```

---

## TASK-013 — Start Fullscreen After OTP

After successful start OTP verification:

```text
OTP_VERIFIED
↓
Request fullscreen
↓
FULLSCREEN_ACTIVE
↓
ANSWERING
```

- [ ] Do not start the controlled assessment before OTP verification.
- [ ] Request fullscreen from a user-initiated interaction where browser rules require it.
- [ ] Handle fullscreen permission failure.
- [ ] Handle browser incompatibility.
- [ ] Show appropriate fallback/error state.

---

## TASK-014 — Fullscreen Guard

Create:

```text
FullscreenGuard
```

Responsibilities:

- [ ] Monitor fullscreen state.
- [ ] Monitor `fullscreenchange`.
- [ ] Monitor `visibilitychange`.
- [ ] Monitor window blur.
- [ ] Monitor window focus.
- [ ] Record suspicious events.
- [ ] Trigger exit confirmation when appropriate.
- [ ] Avoid duplicate dialogs for the same event.
- [ ] Debounce noisy focus/visibility events.

---

# Phase 7 — Tab/Window/Focus Monitoring

## TASK-015 — Visibility Monitoring

Implement:

```js
document.addEventListener("visibilitychange", ...)
```

- [ ] Detect hidden state.
- [ ] Detect visible state.
- [ ] Record `VISIBILITY_HIDDEN`.
- [ ] Record restoration.
- [ ] Trigger exit confirmation where appropriate.
- [ ] Avoid triggering multiple exit dialogs for one event.

---

## TASK-016 — Window Focus Monitoring

Implement:

```js
window.addEventListener("blur", ...)
window.addEventListener("focus", ...)
```

- [ ] Detect focus loss.
- [ ] Record `WINDOW_BLUR`.
- [ ] Detect focus restoration.
- [ ] Record `WINDOW_FOCUS`.
- [ ] Trigger confirmation where appropriate.

---

## TASK-017 — Fullscreen Exit Monitoring

Implement:

```js
document.addEventListener("fullscreenchange", ...)
```

- [ ] Detect unexpected fullscreen exit.
- [ ] Record `FULLSCREEN_EXITED`.
- [ ] Trigger exit confirmation if the question has not been submitted.
- [ ] Do not immediately deduct points.
- [ ] Distinguish expected exit from suspicious exit.

---

## TASK-018 — Keyboard Event Monitoring

Investigate browser-detectable handling for:

```text
Alt + Tab
Ctrl + Tab
Ctrl + W
F11
Escape
```

- [ ] Implement appropriate `keydown` listeners where browser APIs allow.
- [ ] Record relevant detectable events.
- [ ] Trigger confirmation where appropriate.
- [ ] Test behavior in supported browsers.

Important browser limitation:

A normal website cannot reliably intercept or prevent every OS-level shortcut. In particular, browser JavaScript cannot guarantee that `Alt+Tab`, operating-system application switching, Task Manager, process termination, or every browser/OS shortcut can be blocked.

The implementation must therefore use best-effort browser monitoring and must not claim to provide true OS-level kiosk security.

For stronger lockdown requirements, evaluate a dedicated kiosk application or managed-device environment.

---

# Phase 8 — Assessment State Machine

## TASK-019 — Implement Explicit Assessment States

Use an explicit state machine.

Recommended states:

```text
QUESTION_SELECTED
OTP_SENT
OTP_VERIFIED
FULLSCREEN_ACTIVE
ANSWERING
SUBMITTED
COMPLETED
EXIT_REQUESTED
EXIT_OTP_SENT
EXIT_VERIFIED
SESSION_UNRESOLVED
INTERRUPTION_PENALTY_APPLIED
TERMINATED
```

Expected normal flow:

```text
QUESTION_SELECTED
↓
OTP_SENT
↓
OTP_VERIFIED
↓
FULLSCREEN_ACTIVE
↓
ANSWERING
↓
SUBMITTED
↓
COMPLETED
```

Improper exit flow:

```text
ANSWERING
↓
EXIT_REQUESTED
↓
EXIT_OTP_SENT
↓
EXIT_VERIFIED
↓
TERMINATED
```

- [ ] Define allowed transitions.
- [ ] Reject invalid transitions on backend.
- [ ] Do not rely only on frontend state.
- [ ] Persist authoritative state in database.
- [ ] Add state transition tests.

---

# Phase 9 — Exit Confirmation

## TASK-020 — Exit Confirmation Modal

Create reusable:

```text
ExitConfirmationModal
```

Message:

```text
Do you want to exit without submitting the answer?

Your answer will not be submitted.
An OTP will be required to exit.

[Continue Assessment]
[Exit]
```

- [ ] Continue button.
- [ ] Exit button.
- [ ] Disable duplicate actions while processing.
- [ ] Close modal safely.
- [ ] Prevent accidental repeated exit requests.
- [ ] Restore fullscreen when Continue is selected where possible.

---

## TASK-021 — Suspicious Exit Handling

When the user loses focus, switches tabs/windows, or exits fullscreen before submitting:

```text
Suspicious exit detected
↓
Show confirmation
↓
Continue Assessment OR Exit
```

- [ ] Do not automatically deduct points.
- [ ] If Continue is selected, resume assessment.
- [ ] If Exit is selected, initiate exit OTP flow.
- [ ] Record the triggering event.
- [ ] Record the user's final decision.

---

# Phase 10 — Quit Button

## TASK-022 — Add Quit Button

Add a visible:

```text
Quit
```

button to the assessment interface.

Flow:

```text
Quit
↓
Confirmation
↓
Exit
↓
Request exit OTP
↓
Verify OTP
↓
Terminate assessment
↓
-2 points
```

---

## TASK-023 — Quit Confirmation

Display:

```text
Quit Assessment?

You have not submitted this question.
Exiting will require an OTP and deduct 2 points.

[Cancel]
[Quit]
```

- [ ] Cancel returns to assessment.
- [ ] Quit starts exit verification.
- [ ] Do not deduct points merely by clicking Quit.
- [ ] Do not deduct points before successful exit OTP verification.

---


# Phase 10A — Unexpected Browser Closure / Forced Termination

## TASK-025A — Handle Unexpected Browser Disappearance

CodeAira must account for situations where the browser disappears without going through the normal exit flow, including:

```text
Alt + F4
Browser X/Close
Forceful browser termination
Browser process killed
Browser crash
Operating-system termination
Computer shutdown/restart
```

### Core rule

The backend must NOT depend on the frontend to explicitly report that the browser was closed.

The server must maintain the authoritative assessment session state and detect interruption through server-side session/heartbeat information.

When an active assessment loses contact unexpectedly:

```text
ACTIVE / ANSWERING
        ↓
No valid heartbeat / session activity
        ↓
SESSION_UNRESOLVED / POTENTIALLY_INTERRUPTED
```

Do not immediately assume every temporary network interruption is a browser closure.

---

## TASK-025B — Assessment Heartbeat

Implement a periodic assessment heartbeat.

Suggested behavior:

```text
Frontend
   ↓
Periodic heartbeat
   ↓
Backend
   ↓
Update last_seen_at
```

Add to assessment session:

```text
last_seen_at
heartbeat_status
interruption_detected_at
```

- [ ] Send heartbeat periodically while an assessment is active.
- [ ] Backend updates `last_seen_at`.
- [ ] Backend determines whether the session has become stale.
- [ ] Do not rely solely on `visibilitychange`, `blur`, `beforeunload`, or other frontend events.
- [ ] Make heartbeat interval configurable.
- [ ] Make stale-session threshold configurable.
- [ ] Avoid treating a short network interruption as an immediate forced browser closure.

---

## TASK-025C — Mark Potentially Interrupted Sessions

If the backend determines that an active assessment has become stale/unresolved:

```text
status = SESSION_UNRESOLVED
```

Record:

```text
interruption_detected_at
exit_reason = UNEXPECTED_DISAPPEARANCE
```

- [ ] Record an audit event.
- [ ] Preserve the original question ID.
- [ ] Preserve the assessment session ID.
- [ ] Preserve the user's overall score before applying the penalty.
- [ ] Do not depend on the frontend sending a final "browser closed" request.

Important:

The system cannot always distinguish with certainty between:

```text
Browser forcefully closed
Network failure
Computer sleep
Temporary connection loss
Browser crash
```

Therefore the backend should use heartbeat/session timeout logic and explicitly model the result as a potentially interrupted session rather than pretending that the exact cause is always known.

---

## TASK-025D — Unexpected Closure Penalty

If an active unsubmitted assessment is determined to have been unexpectedly terminated/abandoned according to the server-side interruption policy:

```text
Unexpected browser disappearance
        ↓
-3 points
        ↓
Session terminated
```

Rules:

- [ ] Apply exactly `-3`.
- [ ] Apply the penalty server-side.
- [ ] Apply the penalty atomically.
- [ ] Apply the penalty at most once.
- [ ] Store `score_delta = -3`.
- [ ] Store `exit_reason = UNEXPECTED_DISAPPEARANCE`.
- [ ] Record the penalty in the audit log.
- [ ] Do not require the frontend to submit an exit request first.
- [ ] Do not apply both `-3` and the normal unsubmitted-exit `-2` to the same incident.

---

## TASK-025E — Resume Same Question After Unexpected Closure

When the user returns to CodeAira after an unexpected browser disappearance:

```text
Previous session:
SESSION_UNRESOLVED
        ↓
User logs in/returns
        ↓
Detect unresolved assessment
        ↓
Apply/confirm interruption penalty according to server state
        ↓
Resume same question
```

The user must return to the **same question**.

- [ ] Store the question ID in the unresolved session.
- [ ] Detect the unresolved session after login.
- [ ] Do not automatically advance to the next question.
- [ ] Reopen the same question.
- [ ] Create a new active attempt/session only after the interruption has been resolved according to the backend rules.

---

## TASK-025F — Reset Answer After Unexpected Closure

When the user returns after an unexpected browser disappearance:

```text
Same question
+
Previous answer/code = RESET
```

The code submitted/typed before the unexpected closure must NOT be restored as the active answer.

- [ ] Reset code editor to the question's original starter code/empty state.
- [ ] Do not restore the user's unsaved answer.
- [ ] Do not treat the previous answer as submitted.
- [ ] Do not count the previous answer toward completion.
- [ ] Do not preserve unsaved editor state across the interrupted session.
- [ ] The user must solve the same question again from the reset state.

If the platform stores drafts for other purposes, explicitly exclude interrupted-session drafts from automatic restoration.

---

## TASK-025G — Unexpected Closure Return Flow

Implement:

```text
User opens CodeAira
        ↓
Authentication
        ↓
Backend finds unresolved assessment
        ↓
Show:
"Your previous assessment session was unexpectedly interrupted."
        ↓
Apply/confirm -3 according to server-side session state
        ↓
Show same question
        ↓
Reset answer/code
        ↓
Require the normal start/verification flow for the resumed assessment
        ↓
Enter fullscreen
        ↓
Continue solving
```

The exact UX should make clear:

```text
Penalty: -3 points
Question: Same previous question
Answer: Reset
```

---

## TASK-025H — Prevent Duplicate -3 Penalties

The interruption penalty must be idempotent.

Example:

```text
Session becomes unresolved
        ↓
-3 applied
        ↓
Session marked as interruption-penalized
```

If the user:

```text
Refreshes
Logs out
Logs back in
Opens another tab
Retries the request
```

the same interruption must NOT cause another `-3`.

Add a persistent state such as:

```text
interruption_penalty_applied = true
```

or use an equivalent transaction-safe mechanism.

Test:

- [ ] Refresh after interruption.
- [ ] Re-login after interruption.
- [ ] Multiple resume requests.
- [ ] Duplicate API calls.
- [ ] Concurrent resume requests.
- [ ] Backend retry.
- [ ] Network retry.

Expected:

```text
One unexpected interruption → exactly -3 once.
```

---

## TASK-025I — Distinguish Exit Types

CodeAira must distinguish the following cases:

### 1. Completed question → Quit

```text
Question completed
↓
+1 already awarded
↓
User chooses Quit
↓
Normal exit
↓
No OTP
↓
No -2
↓
No -3
```

### 2. Unsubmitted question → Normal Quit / Confirmed Exit

```text
Question active
↓
User chooses Quit/Exit
↓
Exit OTP
↓
OTP verified
↓
-2
↓
Terminate
```

### 3. Browser disappears unexpectedly

```text
Question active
↓
Browser/session disappears
↓
Backend detects unresolved session
↓
-3
↓
Terminate unresolved session
↓
User returns
↓
Same question
↓
Answer reset
```

### 4. Successful completion → Next Question

```text
Question completed
↓
+1
↓
"Do you want to move to the next question?"
↓
Yes
↓
Next question
```

---

## TASK-025J — Do Not Trust Frontend Closure Events

Frontend events such as:

```text
beforeunload
unload
visibilitychange
blur
fullscreenchange
```

may be used as supporting signals, but they must NOT be the authoritative mechanism for determining whether the browser was forcefully closed.

The backend must rely on:

```text
assessment session state
+
last_seen_at / heartbeat
+
server-side timeout/staleness rules
+
persisted audit records
```

The frontend may report an event when possible, but the server must be able to detect an abandoned/unresolved session even when no frontend event arrives.

---

## TASK-025K — Unexpected Closure Tests

Test:

```text
Start question
↓
OTP verified
↓
Fullscreen
↓
Answer partially
↓
Alt + F4
↓
Browser disappears
↓
Backend eventually marks session unresolved
↓
-3 exactly once
↓
User returns
↓
Same question
↓
Answer reset
```

Test:

```text
Start question
↓
Browser process forcefully terminated
↓
No frontend exit request
↓
Backend detects stale session
↓
-3
```

Test:

```text
Start question
↓
Temporary network failure
↓
Heartbeat temporarily unavailable
↓
Connection restored before stale-session threshold
↓
Do NOT incorrectly apply -3
```

Test:

```text
Start question
↓
Browser closes
↓
User returns
↓
Refresh/retry multiple times
↓
Only one -3
↓
Same question
↓
Reset answer
```

Test:

```text
Completed question
↓
+1
↓
Browser closes after completion
↓
No -3
```

Test:

```text
Completed question
↓
+1
↓
User normally quits
↓
No -2
↓
No -3
```

Test:

```text
Unsubmitted question
↓
Normal Quit
↓
Exit OTP
↓
-2
```

The `-2` and `-3` penalties must never both apply to the same exit incident.


# Phase 11 — Exit OTP

## TASK-024 — Request Exit

Endpoint:

```text
POST /api/assessment/:id/request-exit
```

Backend:

- [ ] Verify authentication.
- [ ] Verify session ownership.
- [ ] Verify assessment is active.
- [ ] Verify question is not already submitted.
- [ ] Set state to `EXIT_REQUESTED`.
- [ ] Generate exit OTP.
- [ ] Hash OTP.
- [ ] Store expiry.
- [ ] Store purpose as `EXIT_ASSESSMENT`.
- [ ] Send exit OTP.
- [ ] Set state to `EXIT_OTP_SENT`.
- [ ] Log exit request.
- [ ] Return only non-sensitive information.

---

## TASK-025 — Verify Exit OTP

Endpoint:

```text
POST /api/assessment/:id/verify-exit-otp
```

- [ ] Verify authentication.
- [ ] Verify session ownership.
- [ ] Verify session state.
- [ ] Verify OTP purpose.
- [ ] Check OTP expiry.
- [ ] Check OTP attempt limit.
- [ ] Verify OTP.
- [ ] Confirm question was not successfully submitted.
- [ ] Apply `-2` atomically.
- [ ] Store `score_delta = -2`.
- [ ] Mark session as terminated.
- [ ] Store exit reason.
- [ ] Record `EXIT_OTP_VERIFIED`.
- [ ] Return successful termination response.

---

## TASK-026 — Exit OTP UI

Display:

```text
Exit Verification

Your question has not been submitted.

A verification OTP has been sent to your registered email.

Exiting will deduct 2 points.

[ _ _ _ _ _ _ ]

[Verify Exit]
[Resend OTP]
```

Implement:

- [ ] OTP input.
- [ ] Countdown.
- [ ] Invalid OTP error.
- [ ] Expired OTP error.
- [ ] Maximum attempts error.
- [ ] Resend OTP.
- [ ] Disable verification while request is pending.
- [ ] Handle network errors.

---

# Phase 12 — Code Editor

## TASK-027 — Integrate Code Editor

Possible editor:

```text
Monaco Editor
```

- [ ] Integrate editor.
- [ ] Add syntax highlighting.
- [ ] Add language selection.
- [ ] Add starter code.
- [ ] Add editor settings.
- [ ] Add line numbers.
- [ ] Add basic autocomplete where supported.
- [ ] Preserve code during normal UI state changes.
- [ ] Prevent accidental loss of code during confirmation modals.

---

# Phase 13 — Code Execution

## TASK-028 — Design Code Execution Service

Never execute arbitrary user code directly inside the main backend process.

Use:

```text
Backend
↓
Code Execution Service
↓
Sandbox/container
↓
Execution Result
```

The execution environment must enforce:

- [ ] CPU limit.
- [ ] Memory limit.
- [ ] Execution timeout.
- [ ] Process isolation.
- [ ] Network restrictions.
- [ ] File-system restrictions.
- [ ] Maximum output size.
- [ ] Maximum process count where applicable.

---

## TASK-029 — Run Code API

Create:

```text
POST /api/assessment/:id/run
```

- [ ] Validate session.
- [ ] Validate session ownership.
- [ ] Validate assessment state.
- [ ] Validate language.
- [ ] Validate code.
- [ ] Send code to sandbox.
- [ ] Execute against appropriate test cases.
- [ ] Return output/test results.
- [ ] Handle compilation errors.
- [ ] Handle runtime errors.
- [ ] Handle timeout.
- [ ] Handle memory limit.
- [ ] Do not expose server internals.

---

# Phase 14 — Submission

## TASK-030 — Submit Answer API

Create:

```text
POST /api/assessment/:id/submit
```

- [ ] Validate authentication.
- [ ] Validate session ownership.
- [ ] Validate session state.
- [ ] Validate code.
- [ ] Execute submission tests in sandbox.
- [ ] Determine pass/fail.
- [ ] Store submission.
- [ ] Record submission event.
- [ ] Prevent duplicate successful completion.
- [ ] Mark session submitted only after valid submission.
- [ ] Transition to `SUBMITTED`.
- [ ] Transition to `COMPLETED` when completion criteria are met.

---

## TASK-031 — Submission Rules

Define clearly:

```text
Submitted and correct
→ question completed
→ +1 score
```

If a submission is incorrect:

```text
Question remains active
→ user can continue solving
```

If submission fails due to infrastructure/network error:

```text
Do not apply score penalty.
Do not falsely mark question as completed.
```

---

# Phase 15 — Successful Completion

## TASK-032 — Completion Score

When a question is successfully completed:

```text
score_delta = +1
```

- [ ] Apply score atomically.
- [ ] Store score delta.
- [ ] Prevent duplicate score application.
- [ ] Record score event.
- [ ] Update user overall score.
- [ ] Ensure the same question/session cannot award +1 twice.

---

## TASK-033 — Completion Screen

Display:

```text
Question Completed

+1 point

[Proceed to Next Question]
[Exit Fullscreen]
```

- [ ] Show updated score.
- [ ] Disable duplicate clicks while processing.
- [ ] Preserve completion state.
- [ ] Prevent accidental resubmission.

---


# Phase 15A — Post-Completion Decision Flow

## TASK-032A — Completed Question Decision Flow

After a question is successfully submitted and completed:

```text
Question submitted successfully
        ↓
      +1 mark
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
       Quit     Remain on
       directly completed
       No OTP    question
       No -2
```

### Required behavior

- [ ] Successfully completed question awards exactly `+1`.
- [ ] After the `+1` is awarded, ask:
  - `Do you want to move to the next question?`
- [ ] If the user selects **Yes**:
  - [ ] Create/start the next question flow.
  - [ ] Do not apply any additional score change.
  - [ ] Follow the configured start-OTP policy for the next question.
- [ ] If the user selects **No**:
  - [ ] Ask:
    - `Do you want to quit?`
- [ ] If the user selects **Yes** to quit:
  - [ ] Quit the assessment directly.
  - [ ] Exit fullscreen.
  - [ ] Do NOT require an exit OTP.
  - [ ] Do NOT deduct `-2`.
  - [ ] Preserve the `+1` already awarded.
  - [ ] End the current assessment session as a normally completed session.
- [ ] If the user selects **No** to quit:
  - [ ] Do not terminate the completed session.
  - [ ] Keep the user on the completed-question state or return them to the appropriate completed-question screen.
  - [ ] Do not change the score.

### Important distinction

The `-2` penalty applies ONLY when:

```text
Question is NOT submitted/completed
        ↓
User chooses to exit/quit
        ↓
Exit OTP verified
        ↓
-2 points
```

The `-2` penalty MUST NOT apply when:

```text
Question is successfully completed
        ↓
+1 point
        ↓
User chooses not to proceed
        ↓
User confirms Quit
        ↓
Quit directly
        ↓
No OTP
No -2
```

---

## TASK-032B — Post-Completion UI

Create a post-completion decision modal/screen:

```text
Question Completed

You earned +1 point.

Do you want to move to the next question?

[Yes, Next Question]
[No]
```

If the user selects `No`, display:

```text
Do you want to quit?

[Yes, Quit]
[No, Stay]
```

### UI requirements

- [ ] Clearly display the `+1` earned.
- [ ] Clearly distinguish completed-question exit from unsubmitted-question exit.
- [ ] Do not show an exit-OTP warning for this completed-question quit flow.
- [ ] Do not show a `-2` warning for this completed-question quit flow.
- [ ] Prevent duplicate clicks from awarding multiple `+1` points.
- [ ] Disable buttons while a transition is being processed.
- [ ] Preserve the completed state during network retries.

---

## TASK-032C — Post-Completion Backend State

The backend must distinguish between:

```text
COMPLETED
```

and:

```text
ANSWERING / ACTIVE
```

When the user reaches `COMPLETED`:

- [ ] The `+1` score update must be applied atomically.
- [ ] The session must record `score_delta = +1`.
- [ ] The completion must be idempotent.
- [ ] A completed session must not trigger the unsubmitted exit-OTP flow.
- [ ] A completed session must not receive the `-2` penalty.
- [ ] A completed session can be terminated through the normal completed-quit flow.

---

## TASK-032D — Post-Completion Tests

Test the complete flow:

```text
Correct submission
→ +1
→ Ask "Do you want to move to the next question?"
→ Yes
→ Next question
```

Test:

```text
Correct submission
→ +1
→ Ask "Do you want to move to the next question?"
→ No
→ Ask "Do you want to quit?"
→ Yes
→ Quit
→ No OTP
→ No -2
→ Final score remains +1
```

Test:

```text
Correct submission
→ +1
→ Ask "Do you want to move to the next question?"
→ No
→ Ask "Do you want to quit?"
→ No
→ Remain completed
→ Score remains +1
```

Test duplicate requests:

```text
Submit
→ +1

Retry submit
→ no additional +1
```

Test the distinction:

```text
Completed question + Quit
→ no OTP
→ no -2
```

versus:

```text
Unsubmitted question + Quit
→ exit OTP
→ successful OTP
→ -2
```


# Phase 16 — Normal Exit After Completion

## TASK-034 — Normal Quit After Completion

The completed-question flow is handled by TASK-032A through TASK-032D.

When the user has completed the question and chooses:

```text
Do you want to move to the next question?
→ No
→ Do you want to quit?
→ Yes
```

then:

- [ ] Exit fullscreen.
- [ ] End current assessment session as normally completed.
- [ ] Do not require exit OTP.
- [ ] Do not deduct `-2`.
- [ ] Preserve the `+1` completion score.
- [ ] Return to dashboard/result screen.
- [ ] Record normal completed exit event.

This is different from quitting an unsubmitted question. An unsubmitted question follows the exit-OTP and `-2` penalty flow.

---

# Phase 17 — Proceed to Next Question

## TASK-035 — Next Question Flow

When the user selects:

```text
Proceed to Next Question
```

- [ ] Determine next question.
- [ ] Create a new assessment session.
- [ ] Reset assessment state.
- [ ] Follow configured OTP/start policy.
- [ ] Maintain overall score.
- [ ] Prevent previous question from being counted again.
- [ ] Load next question.
- [ ] Enter the new assessment flow.

---

# Phase 18 — Scoring Integrity

## TASK-036 — Score Rules

Implement exactly:

### Successful completion

```text
Submit/complete question
→ +1
```

### Successful completion + normal exit

```text
Question completed
→ Exit Fullscreen
→ +1
→ No exit OTP
```

### Unsubmitted + verified exit

```text
Question not submitted
→ Exit/Quit
→ Exit OTP
→ Successful verification
→ -2
```

Do not apply both `+1` and `-2` to the same question/session unless a future product requirement explicitly changes this behavior.

---

## TASK-037 — Prevent Duplicate Penalties

Score update must be idempotent.

Example:

```text
Exit OTP verified
↓
score -= 2
↓
session = terminated
```

If the same request is repeated:

```text
score must NOT change again
```

Test:

- [ ] Double-click.
- [ ] Network retry.
- [ ] Duplicate API request.
- [ ] Refresh after successful verification.
- [ ] Multiple browser requests.
- [ ] Race condition between two exit requests.

---

## TASK-038 — Atomic Score Updates

Use a transaction or equivalent atomic database operation.

Required guarantees:

```text
+1 is applied exactly once.
-2 is applied exactly once.
```

The frontend must never directly modify authoritative score.

---

# Phase 19 — Assessment UI

## TASK-039 — Main Assessment Layout

Create:

```text
AssessmentPage
├── Header
│   ├── Question title
│   ├── Timer
│   └── Score
│
├── ProblemPanel
│   ├── Description
│   ├── Examples
│   ├── Constraints
│   └── Test cases
│
├── CodePanel
│   ├── Language selector
│   ├── Editor
│   └── Console
│
└── Footer
    ├── Run
    ├── Submit
    └── Quit
```

---

## TASK-040 — Assessment Header

- [ ] Question title.
- [ ] Difficulty.
- [ ] Timer.
- [ ] User score.
- [ ] Assessment status.
- [ ] Fullscreen status indicator where useful.

---

## TASK-041 — Problem Panel

- [ ] Problem description.
- [ ] Examples.
- [ ] Constraints.
- [ ] Input/output description.
- [ ] Test cases.
- [ ] Scrollable layout.
- [ ] Responsive behavior.

---

## TASK-042 — Code Panel

- [ ] Code editor.
- [ ] Language selector.
- [ ] Run button.
- [ ] Submit button.
- [ ] Output console.
- [ ] Compilation/runtime error display.

---

## TASK-043 — Quit UI

- [ ] Make Quit visible.
- [ ] Keep Quit separate from Submit.
- [ ] Require confirmation.
- [ ] Explain `-2` penalty before OTP verification.
- [ ] Prevent accidental activation.

---

# Phase 20 — OTP UI

## TASK-044 — Start OTP Modal

Display:

```text
Verify Your Email

A 6-digit OTP has been sent to your registered email.

[ _ _ _ _ _ _ ]

[Verify]
[Resend OTP]
```

Implement:

- [ ] OTP input.
- [ ] Countdown.
- [ ] Invalid OTP error.
- [ ] Expired OTP error.
- [ ] Maximum attempts error.
- [ ] Resend OTP.
- [ ] Loading state.
- [ ] Network error state.

---

## TASK-045 — Exit OTP Modal

Display:

```text
Exit Verification

Your question has not been submitted.

A verification OTP has been sent to your registered email.

Exiting will deduct 2 points.

[ _ _ _ _ _ _ ]

[Verify Exit]
[Resend OTP]
```

---

# Phase 21 — Event Logging

## TASK-046 — Assessment Event Logger

Create event types:

```text
ASSESSMENT_STARTED
OTP_SENT
OTP_VERIFIED
FULLSCREEN_ENTERED
FULLSCREEN_EXITED
VISIBILITY_HIDDEN
WINDOW_BLUR
WINDOW_FOCUS
QUIT_CLICKED
EXIT_REQUESTED
EXIT_OTP_SENT
EXIT_OTP_VERIFIED
SUBMISSION_STARTED
SUBMISSION_COMPLETED
ASSESSMENT_COMPLETED
ASSESSMENT_TERMINATED
```

- [ ] Store user ID.
- [ ] Store session ID.
- [ ] Store event type.
- [ ] Store server timestamp.
- [ ] Store metadata.
- [ ] Create events server-side for security-sensitive operations.
- [ ] Do not allow frontend to fabricate score events.

---

## TASK-047 — Event Metadata

Where appropriate record:

```text
browser
operating system
user agent
event source
question_id
session_id
timestamp
```

Do not collect unnecessary personal data.

---

# Phase 22 — Security

## TASK-048 — Backend Authority

The backend must be authoritative for:

- [ ] Score.
- [ ] OTP verification.
- [ ] Assessment state.
- [ ] Submission state.
- [ ] Completion state.
- [ ] Exit penalty.
- [ ] Session ownership.
- [ ] User authorization.

Never trust:

```text
localStorage
sessionStorage
frontend score
frontend completion state
frontend OTP state
frontend assessment state
```

for security-sensitive decisions.

---

## TASK-049 — Authentication Security

- [ ] Use secure password hashing if passwords are used.
- [ ] Use secure sessions/tokens.
- [ ] Use HTTPS in production.
- [ ] Use secure cookies where appropriate.
- [ ] Implement CSRF protection where applicable.
- [ ] Implement session expiration.
- [ ] Implement logout/invalidation.
- [ ] Prevent session fixation.

---

## TASK-050 — OTP Security

- [ ] Cryptographically secure OTP generation.
- [ ] Hash OTP.
- [ ] Expire OTP.
- [ ] Limit attempts.
- [ ] Rate-limit resend.
- [ ] Rate-limit verification.
- [ ] Invalidate OTP after successful use.
- [ ] Separate START and EXIT OTP purposes.
- [ ] Never return OTP through API.
- [ ] Never log plaintext OTP in production.

---

## TASK-051 — API Security

Apply validation and authorization to:

```text
POST /api/auth/login
POST /api/auth/logout
GET  /api/questions
GET  /api/questions/:id
POST /api/assessment/start
POST /api/assessment/verify-start-otp
POST /api/assessment/:id/run
POST /api/assessment/:id/submit
POST /api/assessment/:id/request-exit
POST /api/assessment/:id/verify-exit-otp
GET  /api/assessment/:id
GET  /api/user/profile
GET  /api/user/score
```

- [ ] Validate all inputs.
- [ ] Reject malformed IDs.
- [ ] Reject unauthorized sessions.
- [ ] Apply rate limits.
- [ ] Avoid sensitive information in errors.
- [ ] Log security-relevant failures.

---

# Phase 23 — Code Execution Security

## TASK-052 — Sandbox Security

Never execute user code directly inside the API server.

Implement:

- [ ] Container/process isolation.
- [ ] CPU limit.
- [ ] Memory limit.
- [ ] Execution timeout.
- [ ] Network isolation.
- [ ] Restricted filesystem.
- [ ] Output size limit.
- [ ] Process limit.
- [ ] Cleanup after execution.
- [ ] Prevent access to backend secrets.
- [ ] Prevent access to database credentials.
- [ ] Prevent access to host filesystem.

---

# Phase 24 — Rate Limiting

## TASK-053 — Rate Limit Sensitive APIs

Apply rate limits to:

```text
Login
OTP generation
OTP verification
OTP resend
Assessment start
Code execution
Submission
Exit verification
```

- [ ] Configure per-user limits.
- [ ] Configure IP-based limits where appropriate.
- [ ] Return appropriate errors.
- [ ] Log repeated abuse.

---

# Phase 25 — Error Handling

## TASK-054 — OTP Errors

Handle:

- [ ] OTP expired.
- [ ] Invalid OTP.
- [ ] Too many attempts.
- [ ] Resend rate limit.
- [ ] Email delivery failure.
- [ ] Network failure.

---

## TASK-055 — Assessment Errors

Handle:

- [ ] Session expired.
- [ ] Invalid session.
- [ ] Unauthorized session.
- [ ] Invalid state transition.
- [ ] Duplicate submission.
- [ ] Assessment already completed.
- [ ] Assessment already terminated.
- [ ] Question unavailable.

---

## TASK-056 — Fullscreen Errors

Handle:

- [ ] Fullscreen permission failure.
- [ ] Fullscreen unsupported.
- [ ] Browser restrictions.
- [ ] Unexpected fullscreen exit.
- [ ] User presses Escape.
- [ ] Window loses focus.

---

## TASK-057 — Code Execution Errors

Handle:

- [ ] Compilation error.
- [ ] Runtime error.
- [ ] Timeout.
- [ ] Memory limit.
- [ ] Output limit.
- [ ] Sandbox failure.
- [ ] Execution service unavailable.

Do not apply score penalties because of infrastructure failures.

---

# Phase 26 — Testing

## TASK-058 — Authentication Tests

- [ ] Login success.
- [ ] Login failure.
- [ ] Unauthorized dashboard access.
- [ ] Unauthorized assessment access.
- [ ] Session expiration.
- [ ] Logout.

---

## TASK-059 — OTP Tests

- [ ] Valid start OTP.
- [ ] Invalid start OTP.
- [ ] Expired start OTP.
- [ ] Too many start OTP attempts.
- [ ] Valid exit OTP.
- [ ] Invalid exit OTP.
- [ ] Expired exit OTP.
- [ ] Too many exit OTP attempts.
- [ ] Resend OTP.
- [ ] START OTP cannot be used as EXIT OTP.
- [ ] EXIT OTP cannot be used as START OTP.
- [ ] Used OTP cannot be reused.

---

## TASK-060 — Assessment State Tests

Test:

```text
QUESTION_SELECTED
→ OTP_SENT
→ OTP_VERIFIED
→ FULLSCREEN_ACTIVE
→ ANSWERING
→ SUBMITTED
→ COMPLETED
```

Test:

```text
ANSWERING
→ EXIT_REQUESTED
→ EXIT_OTP_SENT
→ EXIT_VERIFIED
→ TERMINATED
```

Test invalid transitions:

```text
COMPLETED → submit again
TERMINATED → submit
TERMINATED → exit again
EXIT_OTP_SENT → claim completion
```

---

## TASK-061 — Score Tests

Verify:

```text
Completed question → +1
Completed + normal exit → +1
Unsubmitted + verified exit → -2
Duplicate completion → no additional +1
Duplicate exit → only -2 once
```

---

## TASK-062 — Submission Tests

- [ ] Correct answer.
- [ ] Incorrect answer.
- [ ] Compilation failure.
- [ ] Runtime failure.
- [ ] Timeout.
- [ ] Duplicate submission.
- [ ] Submission after termination.
- [ ] Submission after completion.
- [ ] Submission by another user against someone else's session.

---

## TASK-063 — Fullscreen Tests

Test:

- [ ] Enter fullscreen.
- [ ] Exit fullscreen.
- [ ] Visibility change.
- [ ] Window blur.
- [ ] Window focus.
- [ ] User clicks Quit.
- [ ] User switches browser tabs.
- [ ] User attempts supported keyboard shortcuts.
- [ ] User returns to assessment.
- [ ] Browser-specific differences.

---

## TASK-064 — Race Condition Tests

Test concurrent:

- [ ] Two submit requests.
- [ ] Two exit requests.
- [ ] Exit and submit at the same time.
- [ ] Two OTP verification requests.
- [ ] Refresh during score update.
- [ ] Network retry after successful request.

Expected:

```text
One valid completion → maximum +1
One valid exit penalty → maximum -2
```

---

# Phase 27 — Browser Compatibility

## TASK-065 — Browser Testing

Test supported versions of:

- [ ] Chrome.
- [ ] Edge.
- [ ] Firefox.
- [ ] Safari if supported.

Test:

- [ ] Fullscreen API.
- [ ] Visibility API.
- [ ] Focus/blur behavior.
- [ ] Keyboard event behavior.
- [ ] Modal behavior.
- [ ] Code editor.
- [ ] Submission.
- [ ] OTP flow.

Document unsupported behavior.

---

# Phase 28 — Production Hardening

## TASK-066 — Environment Configuration

Create environment variables for:

```text
DATABASE_URL
AUTH_SECRET
EMAIL_API_KEY / SMTP credentials
OTP configuration
CODE_EXECUTION_SERVICE_URL
CODE_EXECUTION_SECRET
FRONTEND_URL
BACKEND_URL
```

- [ ] Never commit `.env`.
- [ ] Maintain `.env.example`.
- [ ] Use separate development/staging/production configuration.
- [ ] Rotate secrets when required.

---

## TASK-067 — HTTPS

- [ ] Configure HTTPS in production.
- [ ] Redirect HTTP to HTTPS.
- [ ] Configure secure cookies.
- [ ] Configure HSTS where appropriate.

---

## TASK-068 — Error Monitoring

Track:

- [ ] API errors.
- [ ] Email failures.
- [ ] Code execution failures.
- [ ] Assessment state errors.
- [ ] OTP failures.
- [ ] Unexpected session termination.
- [ ] Database failures.
- [ ] Security-related failures.

---

## TASK-069 — Audit Dashboard

Create an admin view showing:

```text
User
Question
Session
Start time
Submission time
Exit events
OTP events
Score change
Final status
```

- [ ] Add filters.
- [ ] Add search.
- [ ] Add session detail view.
- [ ] Show event timeline.
- [ ] Protect admin endpoints.

---

# Phase 29 — UX and Accessibility

## TASK-070 — Responsive Design

- [ ] Desktop layout.
- [ ] Large-screen fullscreen layout.
- [ ] Handle unsupported small-screen devices appropriately.
- [ ] Prevent broken editor layout.
- [ ] Ensure modal fits viewport.

---

## TASK-071 — Accessibility

- [ ] Keyboard-accessible normal UI controls.
- [ ] Accessible modal focus management.
- [ ] Clear error messages.
- [ ] Visible focus states.
- [ ] Proper labels for OTP inputs.
- [ ] Appropriate ARIA attributes where required.
- [ ] Avoid relying only on color for status.

---

## TASK-072 — User Feedback

Provide clear states for:

```text
Sending OTP...
OTP sent
Verifying OTP...
OTP verified
Entering fullscreen...
Assessment active
Running code...
Submitting...
Question completed
Exit OTP required
Exit verified
Assessment terminated
```

---

# Phase 30 — Monitoring and Reliability

## TASK-073 — Assessment Recovery

Decide and implement behavior for:

- [ ] Browser refresh.
- [ ] Accidental page reload.
- [ ] Network disconnect.
- [ ] Browser crash.
- [ ] Computer sleep.
- [ ] Backend restart.
- [ ] Code execution service outage.
- [ ] Email provider outage.

The system must not silently apply a `-2` penalty because of infrastructure failure.

---

## TASK-074 — Session Timeout

- [ ] Define assessment session timeout.
- [ ] Warn user before timeout.
- [ ] Handle expired sessions.
- [ ] Ensure expired sessions cannot be submitted.
- [ ] Define whether timeout is penalized separately.
- [ ] Do not invent a timeout penalty unless explicitly specified by product requirements.

---

# Phase 31 — API Documentation

## TASK-075 — Document APIs

Document:

```text
POST /api/auth/login
POST /api/auth/logout

GET /api/questions
GET /api/questions/:id

POST /api/assessment/start
POST /api/assessment/verify-start-otp

POST /api/assessment/:id/run
POST /api/assessment/:id/submit

POST /api/assessment/:id/request-exit
POST /api/assessment/:id/verify-exit-otp

GET /api/assessment/:id
GET /api/user/profile
GET /api/user/score
```

For each API document:

- [ ] Authentication requirements.
- [ ] Request body.
- [ ] Response body.
- [ ] Error responses.
- [ ] Rate limits.
- [ ] Valid state requirements.

---

# Phase 32 — Final Integration

## TASK-076 — Complete Normal Flow

Verify end-to-end:

```text
Login
↓
Question Dashboard
↓
Select Question
↓
Start
↓
Start OTP sent
↓
OTP verified
↓
Fullscreen
↓
Solve
↓
Submit Correctly
↓
Question Completed
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
       Quit     Stay on
       directly completed
       No OTP
       No -2
```

---

## TASK-077 — Complete Next-Question Flow

Verify:

```text
Question Completed
↓
+1
↓
Proceed to Next Question
↓
New Assessment Session
↓
Start OTP
↓
Fullscreen
↓
Solve next question
```

---

## TASK-078 — Complete Improper Exit Flow

Verify:

```text
Start Question
↓
OTP verified
↓
Fullscreen
↓
Solve
↓
No submission
↓
Tab/window/fullscreen exit detected
↓
Confirmation
↓
Exit
↓
Exit OTP sent
↓
Exit OTP verified
↓
-2
↓
Fullscreen/session terminated
↓
Dashboard
```

---

## TASK-079 — Complete Quit Flow

Verify:

```text
Start Question
↓
OTP verified
↓
Fullscreen
↓
Click Quit
↓
Confirmation
↓
Confirm Quit
↓
Exit OTP
↓
OTP verified
↓
-2
↓
Terminate session
```


---

## TASK-079A — Complete Unexpected Browser Closure Flow

Verify:

```text
Start Question
↓
OTP verified
↓
Fullscreen
↓
Answer partially
↓
Browser closes via Alt + F4 / forceful termination / crash
↓
No frontend exit request
↓
Backend heartbeat/session timeout detects unresolved session
↓
-3 exactly once
↓
Session marked interrupted/terminated
↓
User returns to CodeAira
↓
Same question reopened
↓
Previous answer/code reset
↓
Normal resume/start verification flow
↓
Fullscreen
↓
User solves the same question again
```

Verify that:

```text
Completed question + browser closure
→ no -3
```

Verify that:

```text
Temporary network interruption
→ no false -3 if recovered before stale threshold
```

---

# Phase 33 — Final Security Audit

## TASK-080 — Client Manipulation Audit

Attempt to manipulate:

```text
localStorage
sessionStorage
frontend score
frontend assessment state
frontend completion state
frontend OTP state
API payloads
```

Expected:

```text
Server rejects unauthorized state changes.
```

---

## TASK-081 — API Authorization Audit

Attempt:

- [ ] Access another user's session.
- [ ] Submit another user's session.
- [ ] Request exit for another user's session.
- [ ] Verify OTP for another user's session.
- [ ] Modify score directly.
- [ ] Repeat score-changing requests.
- [ ] Skip OTP by changing frontend state.

Expected:

```text
Unauthorized operations are rejected.
```

---

## TASK-082 — OTP Security Audit

Verify:

- [ ] OTP is not present in frontend source.
- [ ] OTP is not returned by API.
- [ ] OTP is not stored plaintext.
- [ ] OTP expires.
- [ ] OTP attempts are limited.
- [ ] OTP resend is rate limited.
- [ ] OTP cannot be reused.
- [ ] Start and exit OTP purposes are separated.

---

# Definition of Done

The platform is considered complete when all of the following are true.

## Authentication

- [ ] User can log in.
- [ ] Registered Gmail/email is available to backend.
- [ ] Unauthenticated users cannot start assessments.

## Questions

- [ ] Questions load.
- [ ] User can select a question.
- [ ] User can start a question.

## Start OTP

- [ ] Start OTP is generated securely.
- [ ] OTP is sent to registered email.
- [ ] OTP is stored securely.
- [ ] OTP is verified server-side.
- [ ] Expired/invalid OTPs are rejected.

## Fullscreen

- [ ] Fullscreen is requested after successful start OTP verification.
- [ ] Fullscreen state is monitored.
- [ ] Unexpected fullscreen exit is detected where browser APIs permit.

## Assessment

- [ ] Code editor works.
- [ ] Code execution works in a sandbox.
- [ ] Submission works.
- [ ] Timer works.
- [ ] Question completion is persisted server-side.

## Suspicious Exit

- [ ] Tab/window/focus events are detected where browser APIs permit.
- [ ] Fullscreen exit is detected.
- [ ] Exit confirmation appears.
- [ ] No automatic penalty occurs before confirmed exit.
- [ ] Browser limitations are documented.

## Quit

- [ ] Quit button exists.
- [ ] Quit requires confirmation.
- [ ] Unsubmitted quit requires exit OTP.
- [ ] Exit OTP clearly explains the `-2` penalty.

## Exit OTP

- [ ] Exit OTP is sent.
- [ ] Exit OTP is verified server-side.
- [ ] Exactly `-2` is applied for a valid unsubmitted exit.
- [ ] Duplicate requests cannot cause additional deductions.
- [ ] Session terminates after successful verification.

## Completion

- [ ] Successful completion gives exactly `+1`.
- [ ] After `+1`, ask whether the user wants to move to the next question.
- [ ] Yes → move to the next question.
- [ ] No → ask whether the user wants to quit.
- [ ] Yes to quit → quit directly.
- [ ] Completed-question quit does not require exit OTP.
- [ ] Completed-question quit does not deduct `-2`.
- [ ] The already-awarded `+1` is preserved.
- [ ] No to quit → remain in the completed-question state.
- [ ] Completed question cannot receive duplicate completion points.

## Unexpected Browser Closure

- [ ] Backend heartbeat/session state detects potentially interrupted sessions.
- [ ] Frontend is not the authoritative source for browser closure.
- [ ] Unexpected active-session termination can result in exactly `-3`.
- [ ] User returns to the same question after unexpected closure.
- [ ] Previous unsaved answer/code is reset.
- [ ] Duplicate `-3` penalties are impossible.
- [ ] Temporary network failures do not automatically cause a false `-3`.

## Security

- [ ] Backend owns assessment state.
- [ ] Backend owns score.
- [ ] OTP is not exposed.
- [ ] User code runs in a sandbox.
- [ ] API authorization is enforced.
- [ ] Rate limiting is enabled.
- [ ] Score changes are atomic and idempotent.
- [ ] Audit events are recorded.

## Testing

- [ ] Unit tests pass.
- [ ] Integration tests pass.
- [ ] State-machine tests pass.
- [ ] Score tests pass.
- [ ] OTP tests pass.
- [ ] Security tests pass.
- [ ] Race-condition tests pass.
- [ ] Browser compatibility tests pass.
- [ ] End-to-end normal flow passes.
- [ ] End-to-end improper-exit flow passes.
- [ ] End-to-end Quit flow passes.

# Product Rule Summary

```text
┌──────────────────────────────────────────────────────────────┐
│                    ASSESSMENT START                          │
├──────────────────────────────────────────────────────────────┤
│ Select Question                                              │
│        ↓                                                     │
│ Start OTP → Verify OTP → Fullscreen → Answer                 │
└──────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────┐
│                    SUCCESSFUL COMPLETION                     │
├──────────────────────────────────────────────────────────────┤
│ Submit Correct Answer                                        │
│        ↓                                                     │
│ +1 Point                                                     │
│        ↓                                                     │
│ ┌─────────────────────┬──────────────────────┐              │
│ │ Proceed Next        │ Exit Fullscreen      │              │
│ │                     │                      │              │
│ │ New Question        │ No Exit OTP          │              │
│ └─────────────────────┴──────────────────────┘              │
└──────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────┐
│                    UNSUBMITTED EXIT                          │
├──────────────────────────────────────────────────────────────┤
│ Tab/Window/Fullscreen Exit OR Quit                           │
│        ↓                                                     │
│ Confirmation                                                 │
│        ↓                                                     │
│ Exit                                                         │
│        ↓                                                     │
│ Exit OTP                                                     │
│        ↓                                                     │
│ Verify OTP                                                   │
│        ↓                                                     │
│ -2 Points                                                    │
│        ↓                                                     │
│ Terminate Session                                            │
└──────────────────────────────────────────────────────────────┘
```

# Exit/Termination Rule Summary

```text
┌──────────────────────────────────────────────────────────────┐
│                 COMPLETED QUESTION                           │
├──────────────────────────────────────────────────────────────┤
│ Submit correctly                                             │
│ ↓                                                            │
│ +1                                                           │
│ ↓                                                            │
│ "Do you want to move to the next question?"                 │
│        │                                                     │
│    YES │ NO                                                  │
│        │  ↓                                                  │
│        │ "Do you want to quit?"                             │
│        │      │                                              │
│        │  YES │ NO                                           │
│        │      ↓                                              │
│        │   Quit directly                                     │
│        │   No OTP                                            │
│        │   No -2                                             │
│        │   No -3                                             │
│        ↓                                                     │
│   Next Question                                              │
└──────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────┐
│                 NORMAL UNSUBMITTED EXIT                      │
├──────────────────────────────────────────────────────────────┤
│ Active question                                              │
│ ↓                                                            │
│ Quit / confirmed exit                                        │
│ ↓                                                            │
│ Exit OTP                                                     │
│ ↓                                                            │
│ Verify                                                       │
│ ↓                                                            │
│ -2                                                           │
│ ↓                                                            │
│ Terminate                                                    │
└──────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────┐
│             UNEXPECTED BROWSER DISAPPEARANCE                 │
├──────────────────────────────────────────────────────────────┤
│ Active/unsubmitted question                                  │
│ ↓                                                            │
│ Browser disappears / process terminates / crash              │
│ ↓                                                            │
│ Backend detects stale/unresolved session                     │
│ ↓                                                            │
│ -3 exactly once                                              │
│ ↓                                                            │
│ Session marked interrupted                                   │
│ ↓                                                            │
│ User returns                                                  │
│ ↓                                                            │
│ Same question                                                │
│ ↓                                                            │
│ Previous answer reset                                        │
│ ↓                                                            │
│ Resume through normal assessment verification flow           │
└──────────────────────────────────────────────────────────────┘
```

# Important Browser Security Constraint

This application is a browser-based assessment system, not a true operating-system kiosk.

JavaScript can detect browser-level signals such as:

```text
visibilitychange
blur/focus
fullscreenchange
some keyboard events
```

but it cannot guarantee prevention or detection of every operating-system action.

It cannot reliably block or identify every instance of:

```text
Alt + Tab
Task Manager
OS-level application switching
Process termination
Operating-system shortcuts
Browser process termination
Computer shutdown/restart
```

Therefore the implementation should provide best-effort browser monitoring, confirmation dialogs, server-side assessment state, OTP-based exit verification, and audit logging.

If strict lockdown is required, use a dedicated kiosk application, managed browser, or managed-device solution in addition to this web application.
