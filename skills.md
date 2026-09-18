# CodeAira — skills.md

## 1. Purpose

This file defines the technical skills, technology stack, project structure, development conventions, and implementation guidance for CodeAira.

The agent must use this file together with:

- `AGENTS.md` — agent behavior and project rules
- `tasks.md` — task-by-task implementation roadmap
- `history.txt` — append-only development change history

Priority:

```text
User instructions
↓
AGENTS.md
↓
tasks.md
↓
skills.md
↓
Implementation convenience
```

Do not use `skills.md` to override explicit requirements in `tasks.md` or `AGENTS.md`.

---

# 2. CodeAira Technology Stack

## Frontend

Use:

- React
- Vite
- TypeScript
- React Router
- Tailwind CSS
- Monaco Editor

Recommended frontend structure:

```text
React + TypeScript + Vite
```

### Why TypeScript

Use TypeScript instead of plain JavaScript for:

- Type-safe API responses
- Assessment/session state
- Question data
- Authentication state
- Score data
- Editor state
- Better refactoring
- Easier maintenance as CodeAira grows

---

# 3. Backend

Use:

- Node.js
- Express.js
- TypeScript

Recommended:

```text
Node.js
└── Express.js
    └── TypeScript
```

The backend must be responsible for:

- Authentication/session validation
- OTP verification
- Assessment sessions
- Question retrieval
- Code submission
- Score calculation
- Penalties
- Heartbeats
- Interruption detection
- Session recovery
- Database access
- Authorization
- Security-sensitive operations

The frontend must not be trusted for score or assessment-state decisions.

---

# 4. Database

Use:

- MongoDB
- Mongoose

MongoDB should store:

- Users
- Questions
- Test cases
- Assessment sessions
- Submissions
- Scores
- OTP/session verification metadata
- Audit events
- User progress

Use Mongoose schemas/models to keep database access structured.

---

# 5. Authentication

## Recommended Authentication Architecture

Use:

**Auth.js** for authentication where it fits the application's authentication boundary, with the registered Gmail/Google identity used to establish the user's account.

However, CodeAira's assessment OTP is a separate security mechanism and must remain under the CodeAira backend's control.

Recommended conceptual flow:

```text
User
 ↓
Google/Gmail Authentication
 ↓
CodeAira Account
 ↓
Select Assessment Question
 ↓
CodeAira sends assessment OTP
 ↓
User enters OTP
 ↓
Backend verifies OTP
 ↓
Assessment session becomes active
```

Do NOT treat Google login alone as the assessment-start OTP.

The assessment OTP should be:

- Short-lived
- Single-use
- Rate-limited
- Server-verified
- Bound to the appropriate user/session
- Invalidated after successful use

Never expose the OTP in frontend code.

---

# 6. Authentication Decision

The preferred setup is:

```text
Google OAuth / Auth.js
        +
CodeAira server-side assessment OTP
```

Use Google authentication to establish identity.

Use CodeAira OTP to authorize the sensitive assessment transition.

If Auth.js integration creates significant architectural complications with the Express backend, STOP and ask the user before replacing it with another authentication architecture.

Possible alternatives that may be considered after user approval:

- Better Auth
- Clerk
- Auth0
- Firebase Authentication
- Custom session authentication

Do not introduce a third-party authentication provider without checking its:

- Pricing
- Email/Google support
- Session model
- Data handling
- Backend compatibility
- Deployment requirements

---

# 7. Email / OTP Service

The application needs an email provider for assessment OTP delivery.

The architecture should be:

```text
Express Backend
      ↓
OTP Service
      ↓
Email Provider
      ↓
Registered User Email
```

The exact provider can be selected later.

Possible providers include:

- Resend
- Amazon SES
- SendGrid
- Postmark

Do not hard-code a provider-specific architecture before checking project requirements and cost.

If selecting a provider becomes a significant project decision, ask the user first.

---

# 8. Code Execution

CodeAira must NOT execute arbitrary user code directly inside the main Express.js server process.

Recommended architecture:

```text
React
 ↓
Express API
 ↓
Submission Service
 ↓
Isolated Code Runner
 ↓
Execution Result
 ↓
Express API
 ↓
React
```

The code runner should be isolated from the application server.

Potential technologies:

- Docker containers
- Sandboxed execution workers
- Isolated worker processes
- Dedicated code execution service

For an MVP, Docker-based isolated execution is a reasonable architecture.

The execution environment must have:

- CPU limits
- Memory limits
- Execution time limits
- Process limits
- Network restrictions
- Filesystem restrictions
- Automatic cleanup

Never execute arbitrary submitted code with unrestricted server privileges.

---

# 9. Frontend Folder Structure

Recommended structure:

```text
CodeAira/
│
├── frontend/
│   ├── public/
│   │
│   ├── src/
│   │   ├── assets/
│   │   │
│   │   ├── components/
│   │   │   ├── common/
│   │   │   ├── auth/
│   │   │   ├── assessment/
│   │   │   ├── editor/
│   │   │   ├── questions/
│   │   │   ├── score/
│   │   │   └── layout/
│   │   │
│   │   ├── pages/
│   │   │   ├── Home/
│   │   │   ├── Login/
│   │   │   ├── Dashboard/
│   │   │   ├── Questions/
│   │   │   ├── Assessment/
│   │   │   ├── Results/
│   │   │   └── Profile/
│   │   │
│   │   ├── hooks/
│   │   │
│   │   ├── services/
│   │   │   ├── api.ts
│   │   │   ├── auth.ts
│   │   │   ├── assessment.ts
│   │   │   ├── questions.ts
│   │   │   └── submissions.ts
│   │   │
│   │   ├── store/
│   │   │
│   │   ├── types/
│   │   │
│   │   ├── utils/
│   │   │
│   │   ├── routes/
│   │   │
│   │   ├── App.tsx
│   │   └── main.tsx
│   │
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── .env.example
```

---

# 10. Backend Folder Structure

Recommended:

```text
CodeAira/
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   ├── database.ts
│   │   │   ├── environment.ts
│   │   │   └── auth.ts
│   │   │
│   │   ├── controllers/
│   │   │   ├── auth.controller.ts
│   │   │   ├── question.controller.ts
│   │   │   ├── assessment.controller.ts
│   │   │   ├── submission.controller.ts
│   │   │   └── user.controller.ts
│   │   │
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── question.routes.ts
│   │   │   ├── assessment.routes.ts
│   │   │   ├── submission.routes.ts
│   │   │   └── user.routes.ts
│   │   │
│   │   ├── models/
│   │   │   ├── User.ts
│   │   │   ├── Question.ts
│   │   │   ├── AssessmentSession.ts
│   │   │   ├── Submission.ts
│   │   │   ├── OTP.ts
│   │   │   └── AuditEvent.ts
│   │   │
│   │   ├── services/
│   │   │   ├── auth.service.ts
│   │   │   ├── otp.service.ts
│   │   │   ├── assessment.service.ts
│   │   │   ├── question.service.ts
│   │   │   ├── submission.service.ts
│   │   │   ├── scoring.service.ts
│   │   │   ├── heartbeat.service.ts
│   │   │   ├── interruption.service.ts
│   │   │   └── email.service.ts
│   │   │
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts
│   │   │   ├── error.middleware.ts
│   │   │   ├── rateLimit.middleware.ts
│   │   │   └── validation.middleware.ts
│   │   │
│   │   ├── validators/
│   │   │
│   │   ├── utils/
│   │   │
│   │   ├── types/
│   │   │
│   │   ├── workers/
│   │   │   └── codeRunner.worker.ts
│   │   │
│   │   ├── app.ts
│   │   └── server.ts
│   │
│   ├── tests/
│   │
│   ├── package.json
│   ├── tsconfig.json
│   └── .env.example
```

---

# 11. Root Folder Structure

Recommended final structure:

```text
CodeAira/
│
├── frontend/
├── backend/
├── runner/
│
├── docs/
│
├── tests/
│
├── AGENTS.md
├── tasks.md
├── skills.md
├── history.txt
├── README.md
├── .gitignore
├── .env.example
├── docker-compose.yml
└── package.json
```

The exact structure may evolve as tasks are implemented.

Do not reorganize the project significantly without checking `tasks.md` and asking the user if the change is architectural.

---

# 12. Assessment State Architecture

Assessment state should be represented explicitly.

Example:

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

Possible exit/interruption states:

```text
EXIT_REQUESTED
EXIT_OTP_SENT
EXIT_VERIFIED
SESSION_UNRESOLVED
INTERRUPTION_PENALTY_APPLIED
TERMINATED
```

The backend owns these transitions.

The frontend displays state; it does not determine authoritative state.

---

# 13. Assessment Session Model

A session should contain information similar to:

```text
AssessmentSession
├── userId
├── questionId
├── status
├── startedAt
├── submittedAt
├── endedAt
├── otpVerified
├── exitReason
├── scoreDelta
├── lastSeenAt
├── heartbeatStatus
├── interruptionDetectedAt
├── interruptionPenaltyApplied
├── createdAt
└── updatedAt
```

Do not store more sensitive information than required.

---

# 14. Question Model

A question should support fields similar to:

```text
Question
├── title
├── description
├── difficulty
├── constraints
├── examples
├── starterCode
├── supportedLanguages
├── testCases
├── expectedOutput
├── tags
├── timeLimit
├── memoryLimit
└── metadata
```

Private test cases should never be exposed to the frontend.

---

# 15. Submission Model

A submission should record:

```text
Submission
├── userId
├── questionId
├── assessmentSessionId
├── language
├── code
├── status
├── testResults
├── executionTime
├── memoryUsed
├── submittedAt
└── metadata
```

Sensitive execution details should be protected.

---

# 16. Score Architecture

Never calculate authoritative score only in React.

Correct architecture:

```text
React
 ↓
Submit Answer
 ↓
Express
 ↓
Validate Session
 ↓
Validate Submission
 ↓
Determine Result
 ↓
Update Score Atomically
 ↓
Return Result
```

Score mutations:

```text
Completion → +1
Normal unsubmitted exit → -2
Unexpected browser disappearance → -3
```

Prevent duplicate score changes using transaction/idempotency mechanisms.

---

# 17. Fullscreen Implementation

Frontend may use the browser Fullscreen API.

React should manage:

- Entering fullscreen
- Detecting fullscreen changes
- Detecting visibility changes
- Detecting focus changes
- Showing warnings
- Sending heartbeat/session signals

However:

```text
Frontend detection ≠ authoritative security
```

Backend session state remains authoritative.

Do not claim that CodeAira can completely prevent:

- Alt+Tab
- Alt+F4
- Browser close
- Browser crashes
- OS termination
- Computer shutdown

---

# 18. Heartbeat

During an active assessment:

```text
React
 ↓
Heartbeat
 ↓
Express
 ↓
Update lastSeenAt
```

The backend determines whether the session has become stale.

Suggested configurable values:

```text
HEARTBEAT_INTERVAL
SESSION_STALE_THRESHOLD
```

Do not hard-code these values throughout the application.

---

# 19. Unexpected Closure

The backend must be capable of detecting a potentially interrupted session without requiring a final frontend request.

Use:

```text
lastSeenAt
+
heartbeat
+
session state
+
server-side timeout
```

If the session becomes unresolved:

```text
SESSION_UNRESOLVED
```

For an active/unsubmitted question, the configured interruption policy results in:

```text
-3
```

When the user returns:

```text
Same question
+
Answer reset
```

The interruption penalty must be idempotent.

---

# 20. API Organization

Recommended API prefix:

```text
/api/v1
```

Example:

```text
POST   /api/v1/auth/otp/request
POST   /api/v1/auth/otp/verify

GET    /api/v1/questions
GET    /api/v1/questions/:id

POST   /api/v1/assessments
GET    /api/v1/assessments/:id
POST   /api/v1/assessments/:id/heartbeat
POST   /api/v1/assessments/:id/submit
POST   /api/v1/assessments/:id/exit
POST   /api/v1/assessments/:id/resume

POST   /api/v1/submissions
GET    /api/v1/submissions/:id

GET    /api/v1/users/me
GET    /api/v1/users/me/score
```

The exact API should follow the implementation requirements in `tasks.md`.

---

# 21. Validation

Validate input at the backend boundary.

Use a schema validation library such as:

- Zod
- Joi
- express-validator

Do not assume frontend validation is sufficient.

Validate:

- IDs
- OTPs
- Question IDs
- Submission payloads
- Language
- Code size
- Assessment session IDs
- Pagination
- User input

---

# 22. Error Handling

Use centralized Express error handling.

Recommended:

```text
Request
 ↓
Route
 ↓
Controller
 ↓
Service
 ↓
Error
 ↓
Central error middleware
 ↓
Consistent API response
```

Do not expose:

- Stack traces
- Database errors
- Internal file paths
- Secrets
- OTPs
- Internal service details

in production responses.

---

# 23. Environment Variables

Use environment variables for:

```text
MONGODB_URI
AUTH_SECRET
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
EMAIL_API_KEY
EMAIL_FROM
SESSION_SECRET
CODE_RUNNER_URL
NODE_ENV
PORT
FRONTEND_URL
```

Never commit actual secrets.

Commit:

```text
.env.example
```

Do not commit:

```text
.env
```

---

# 24. Frontend State Management

Start with React state/context where sufficient.

Introduce a dedicated state-management library only when application complexity requires it.

Potential options:

- Zustand
- Redux Toolkit

Do not add state-management infrastructure without a clear need.

Assessment state should have a clearly defined source of truth.

---

# 25. Data Fetching

Use a dedicated API/data-fetching layer.

Possible choice:

- TanStack Query

Use it for:

- Questions
- User data
- Assessment state
- Submission status
- Results

Do not scatter raw `fetch()` calls throughout UI components.

---

# 26. Code Editor

Use Monaco Editor for the coding interface.

The editor should support:

- Syntax highlighting
- Language selection
- Starter code
- Code reset
- Submission
- Read-only mode when appropriate
- Error output
- Test results
- Keyboard shortcuts

The editor's local state must not be treated as proof of submission.

---

# 27. Component Rules

Prefer small, focused React components.

Avoid:

```text
One giant AssessmentPage.tsx
```

Prefer:

```text
AssessmentPage
├── AssessmentHeader
├── QuestionPanel
├── CodeEditor
├── TestResultPanel
├── AssessmentTimer
├── FullscreenGuard
├── ExitConfirmationModal
└── CompletionModal
```

Keep business logic out of presentational components when practical.

---

# 28. Backend Architecture Rules

Prefer:

```text
Route
 ↓
Controller
 ↓
Service
 ↓
Model/Repository
```

Do not place all logic in Express route handlers.

Avoid:

```typescript
app.post("/submit", async (req, res) => {
    // 500 lines of business logic
});
```

Prefer dedicated services.

---

# 29. Security Skills Required

The implementation agent should understand:

- OAuth
- Sessions
- Cookies
- CSRF
- CORS
- JWT/session tradeoffs
- Rate limiting
- Input validation
- Password/OTP security
- Authorization
- MongoDB security
- Secure headers
- Code sandboxing
- SSRF
- Command injection
- Container isolation
- Resource exhaustion
- Race conditions
- Idempotency

For security-critical decisions, ask the user before changing the architecture.

---

# 30. Testing Structure

Frontend:

```text
frontend/
└── src/
    └── ...
```

Tests can be organized as:

```text
frontend/
└── tests/
```

Backend:

```text
backend/
└── tests/
    ├── unit/
    ├── integration/
    ├── auth/
    ├── assessment/
    ├── scoring/
    └── interruption/
```

Important assessment tests:

```text
completion → +1
completed quit → no OTP, no penalty
normal unsubmitted exit → OTP → -2
unexpected disappearance → -3
duplicate interruption → only one -3
resume → same question
resume → answer reset
temporary network loss → no false penalty
```

---

# 31. Git Skills

Use Git for version control.

Recommended branch types:

```text
main
develop
feature/*
fix/*
refactor/*
```

Commit messages should be meaningful.

Examples:

```text
feat: add assessment heartbeat
feat: implement question submission
fix: prevent duplicate completion score
fix: reset interrupted assessment answer
test: add interruption penalty tests
```

Do not make large unrelated commits.

Do not rewrite shared history without user approval.

---

# 32. Documentation

Maintain:

```text
README.md
AGENTS.md
tasks.md
skills.md
history.txt
```

README should explain:

- Project purpose
- Features
- Tech stack
- Setup
- Environment variables
- Development commands
- Testing
- Deployment

Do not duplicate the complete task roadmap into README.

---

# 33. Local Development

Recommended setup:

```text
Frontend:
npm install
npm run dev

Backend:
npm install
npm run dev
```

The exact commands depend on the final package configuration.

Use separate environment configuration for development and production.

---

# 34. Docker

Docker may be used for:

- MongoDB development
- Code execution
- Backend services
- Supporting infrastructure

Do not put untrusted submitted code in the same unrestricted container/process as the main application.

For code execution, use a dedicated restricted execution environment.

---

# 35. Performance

Pay attention to:

- Database indexes
- API response size
- Code execution limits
- Question loading
- Submission latency
- Heartbeat traffic
- MongoDB query performance
- React unnecessary re-renders

Do not prematurely optimize.

Measure before making major performance changes.

---

# 36. Observability

Important backend events should be logged.

Examples:

```text
AUTH_SUCCESS
AUTH_FAILURE
OTP_REQUESTED
OTP_VERIFIED
OTP_FAILED
ASSESSMENT_STARTED
ASSESSMENT_SUBMITTED
ASSESSMENT_COMPLETED
NORMAL_EXIT
EXIT_OTP_VERIFIED
SESSION_UNRESOLVED
INTERRUPTION_PENALTY_APPLIED
ASSESSMENT_RESUMED
CODE_EXECUTION_FAILED
```

Do not log:

- OTP values
- Passwords
- Authentication secrets
- API keys
- Sensitive user data

---

# 37. Audit History

Assessment-sensitive actions should have audit records.

Example:

```text
AuditEvent
├── userId
├── assessmentSessionId
├── eventType
├── timestamp
├── metadata
└── requestId
```

This helps investigate:

- Duplicate scoring
- Unexpected interruptions
- OTP issues
- Session state transitions
- Submission problems

---

# 38. What the Agent Should Do

The agent SHOULD:

- Read `AGENTS.md`.
- Read `tasks.md`.
- Read `skills.md`.
- Find the first incomplete task.
- Inspect existing code before editing.
- Follow the existing architecture.
- Implement one task at a time.
- Run appropriate tests.
- Verify behavior.
- Update `history.txt`.
- Update `tasks.md`.
- Keep changes focused.
- Ask before significant decisions.
- Prefer secure server-side logic.
- Preserve existing functionality.
- Explain blockers clearly.

---

# 39. What the Agent Should NOT Do

The agent must NOT:

- Ignore `tasks.md`.
- Skip tasks without permission.
- Implement multiple unrelated tasks together.
- Invent product requirements.
- Change scoring rules without approval.
- Change authentication architecture without approval.
- Replace MongoDB without approval.
- Replace React without approval.
- Replace Express without approval.
- Delete user data.
- Reset the database.
- Commit secrets.
- Trust frontend score values.
- Trust frontend assessment state.
- Execute arbitrary code directly on the application server.
- Assume fullscreen is unbreakable.
- Claim browser closure can always be detected directly.
- Apply duplicate penalties.
- Restore an interrupted answer when the requirement says it must reset.
- Modify old `history.txt` entries to hide changes.
- Mark a task complete without verification.

---

# 40. When to Ask the User

Ask the user before:

```text
Changing architecture
Changing technology
Changing authentication
Changing scoring
Changing penalties
Changing database schema significantly
Changing task order
Adding major dependencies
Adding paid services
Changing security model
Deleting functionality
Changing assessment behavior
Changing resume behavior
Changing OTP behavior
Changing code execution architecture
```

Do not ask for approval for trivial implementation details already determined by `tasks.md`.

---

# 41. Working With `tasks.md`

The agent must continuously use this loop:

```text
Read tasks.md
      ↓
Find first incomplete task
      ↓
Check dependencies
      ↓
Read relevant files
      ↓
Implement task
      ↓
Test
      ↓
Verify
      ↓
Append to history.txt
      ↓
Mark task complete
      ↓
Repeat
```

If the task cannot be completed:

```text
Do not mark complete.
Record the blocker.
Ask the user if a significant decision is required.
```

---

# 42. Working With `history.txt`

Every meaningful implementation must create an entry.

Minimum information:

```text
Date:
Task:
Files edited:
Changes:
Reason:
Verification:
Notes:
```

Never erase history.

Never falsify verification.

If a test fails, record that it failed.

If a task is partially complete, record it as partial rather than complete.

---

# 43. Recommended Skills for the Developer/Agent

The implementation requires familiarity with:

### Frontend

- React
- TypeScript
- Vite
- React Router
- Tailwind CSS
- Monaco Editor
- Browser Fullscreen API
- Page Visibility API
- Browser lifecycle events
- WebSockets or polling/heartbeat mechanisms

### Backend

- Node.js
- Express.js
- TypeScript
- REST APIs
- Middleware
- Authentication
- Authorization
- Sessions
- OTP systems
- Rate limiting
- Validation

### Database

- MongoDB
- Mongoose
- Indexes
- Transactions
- Atomic updates
- Query optimization

### Security

- OAuth
- Google authentication
- Email OTP
- Secure cookies
- CORS
- CSRF
- Input validation
- Code sandboxing
- Container isolation
- Rate limiting
- Idempotency

### DevOps

- Git
- Docker
- Environment variables
- CI/CD
- Logging
- Deployment

---

# 44. Final Architecture

The intended high-level architecture is:

```text
                         CODEAIRA
                            │
          ┌─────────────────┴─────────────────┐
          │                                   │
      React Frontend                    Express Backend
          │                                   │
          │                              Auth / OTP
          │                                   │
          │                              Assessment
          │                                   │
          │                              Questions
          │                                   │
          │                              Submissions
          │                                   │
          │                              Scoring
          │                                   │
          │                              Heartbeats
          │                                   │
          │                         Interruption Detection
          │                                   │
          │                                   ▼
          │                              MongoDB
          │
          ▼
     Monaco Editor
          │
          │ Submit
          ▼
     Express API
          │
          ▼
   Isolated Code Runner
          │
          ▼
     Execution Result
```

---

# 45. Core Principle

CodeAira should be built as a **server-authoritative assessment platform**.

The frontend is responsible for:

```text
UI
Editor
Fullscreen requests
User interaction
Display
```

The backend is responsible for:

```text
Identity
Authentication
OTP
Authorization
Assessment state
Submission state
Score
Penalties
Session validity
Heartbeat state
Interruption handling
Resume state
```

The database is responsible for persistent state.

The code runner is responsible for isolated execution.

The agent must follow `tasks.md` task-by-task and use `AGENTS.md` to determine how development decisions should be made.
