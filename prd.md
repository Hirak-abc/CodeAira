# CodeAira — Product Requirements Document (PRD)

## 1. Product Overview

**Product Name:** CodeAira

CodeAira is a coding assessment platform inspired by LeetCode. It allows users to solve programming questions in a controlled assessment environment and track their performance.

The product is designed for people who want to practice coding, prepare for interviews, or assess their programming skills.

---

## 2. Target Users

CodeAira primarily targets:

### 1. Students

Students who want to:

- Practice programming
- Prepare for coding exams
- Improve problem-solving skills
- Track their coding performance
- Test themselves under assessment conditions

### 2. Interview Preparation Users

People preparing for:

- Software engineering interviews
- Coding interviews
- Technical assessments
- Placement tests

They can use CodeAira to practice solving questions under controlled conditions.

### 3. Self-Assessment Users

People who want to:

- Test their current programming ability
- Identify their strengths and weaknesses
- Challenge themselves
- Measure their performance over time

---

# 3. Problem Statement

Many coding platforms focus primarily on solving individual problems.

CodeAira aims to provide an additional **assessment-oriented experience** where users solve questions under a controlled session with:

- Fullscreen mode
- OTP-based exit verification
- Session tracking
- Scoring
- Interruption handling
- Question resumption

The objective is to make coding practice closer to a structured assessment experience.

---

# 4. Product Goals

CodeAira should:

1. Provide a simple coding-question experience.
2. Allow users to solve programming problems using an online editor.
3. Provide an assessment mode with fullscreen behavior.
4. Verify assessment entry using an OTP sent to the user's registered email.
5. Discourage intentionally leaving an active assessment without submitting.
6. Handle unexpected browser/session interruptions.
7. Track assessment scores.
8. Allow users to continue to another question after successful completion.
9. Allow users to quit after completing a question without an additional penalty.
10. Allow users to resume an interrupted question according to the assessment rules.

---

# 5. Core User Flow

```text
User
 ↓
Login
 ↓
Select Question
 ↓
Assessment OTP sent to registered email
 ↓
Enter OTP
 ↓
OTP verified
 ↓
Fullscreen assessment starts
 ↓
Solve question
 ↓
Submit
 ↓
+1
 ↓
Move to next question?
 ├── Yes → Next question
 └── No
       ↓
     Quit?
       ├── Yes → Exit directly
       └── No → Continue
```

---

# 6. Authentication

Users must be authenticated before starting an assessment.

The product should support authentication using a suitable authentication provider, with the user's registered email available for assessment OTP delivery.

The assessment OTP is separate from normal login authentication.

### Assessment OTP

The OTP is used when:

- Starting the protected assessment session
- Leaving an unsubmitted question

OTP requirements:

- Sent to the user's registered email
- Short-lived
- Single-use
- Rate-limited
- Verified by the backend

---

# 7. Assessment Mode

When the user successfully verifies the assessment OTP:

```text
OTP verified
↓
Enter fullscreen
↓
Assessment begins
```

The assessment interface should contain:

- Question statement
- Code editor
- Language selector
- Run/test functionality
- Submit button
- Timer where applicable
- Score/session information
- Quit control

---

# 8. Fullscreen Requirements

The assessment should request browser fullscreen after successful OTP verification.

The application should monitor relevant browser signals, including:

- Fullscreen changes
- Tab visibility
- Window focus
- Browser lifecycle events

However, fullscreen is not considered an absolute security mechanism.

The product must account for situations such as:

- Alt+Tab
- Ctrl+Tab
- Alt+F4
- Browser termination
- Browser crashes
- Computer shutdown
- Other unexpected disappearance

The backend must remain authoritative for assessment state.

---

# 9. Completed Question Behavior

When the user successfully submits a question:

```text
Submission accepted
↓
+1 score
↓
Ask:
"Do you want to move to the next question?"
```

### User selects Yes

Move to the next question.

### User selects No

Ask:

```text
"Do you want to quit?"
```

If the user selects Yes:

```text
Exit assessment
↓
Exit fullscreen
↓
No OTP required
↓
No additional penalty
```

The `+1` already earned remains.

Therefore:

```text
Completed question
+
Quit
=
+1
=
No OTP
=
No -2
=
No -3
```

---

# 10. Unsubmitted Question Exit

If the user wants to leave an active question without submitting it:

```text
Active question
↓
User chooses exit/quit
↓
OTP sent to registered email
↓
User enters OTP
↓
Backend verifies OTP
↓
-2 score
↓
Exit assessment
```

The OTP is required specifically to authorize a normal exit from an unsubmitted question.

If OTP verification fails, the normal OTP-authorized exit must not occur.

---

# 11. Quit Button

The assessment must provide a visible **Quit** button.

Its behavior depends on question state.

### If the question is completed

```text
Quit
↓
Exit directly
↓
No OTP
↓
No additional penalty
```

### If the question is unsubmitted

```text
Quit
↓
OTP verification
↓
Successful verification
↓
-2
↓
Exit
```

---

# 12. Interruption Warning

If the browser detects a potential attempt to leave the assessment context, the UI may display a warning such as:

```text
"Do you want to exit without submitting the answer?"
```

Examples of relevant browser signals include:

- Tab change
- Window focus loss
- Fullscreen exit

The frontend may warn the user, but it must not be treated as the authoritative source for session or scoring decisions.

---

# 13. Unexpected Browser Closure

If the browser disappears unexpectedly while the user is solving an unsubmitted question, CodeAira should mark the session as potentially interrupted.

Examples:

- Alt+F4
- Forceful browser close
- Browser crash
- OS-level termination
- Computer shutdown

The backend must not rely on the frontend sending a final "browser closed" message.

Instead, the backend should use mechanisms such as:

```text
Heartbeat
+
lastSeenAt
+
server-side session state
+
stale-session detection
```

---

# 14. Unexpected Interruption Penalty

If an active/unsubmitted session is determined to have been unexpectedly interrupted:

```text
Interruption
↓
-3 score
```

The penalty must be applied only once for the same interruption.

The system must prevent duplicate `-3` penalties caused by:

- Multiple frontend events
- Repeated requests
- Retries
- Race conditions

---

# 15. Resume After Unexpected Closure

When the user returns after an unexpected interruption:

```text
User returns
↓
Find interrupted session
↓
Apply/confirm -3 exactly once
↓
Return to the same question
↓
Reset the answer/code
```

The user should not be moved to the next question.

The previous answer should not be restored when the interruption rule requires the answer to be reset.

---

# 16. Scoring Rules

The authoritative scoring rules are:

| Situation | Score |
|---|---:|
| Successfully completed question | +1 |
| Quit after completing question | No additional penalty |
| Exit unsubmitted question using OTP | -2 |
| Unexpected browser/session interruption | -3 |

Score calculations must be performed by the backend.

The frontend must never be trusted to submit an arbitrary score value.

---

# 17. Question Experience

Each coding question should provide:

- Title
- Problem description
- Examples
- Constraints
- Input/output requirements
- Difficulty
- Tags where applicable
- Starter code
- Supported programming languages
- Test cases
- Run/test functionality
- Submit functionality

Private test cases must remain protected from the frontend.

---

# 18. Code Editor

CodeAira should provide an online coding editor using Monaco Editor or an equivalent editor.

The editor should support:

- Syntax highlighting
- Multiple languages
- Starter code
- Code editing
- Run/test
- Submit
- Reset
- Error/output display

---

# 19. Code Execution

User code must execute in an isolated environment.

The main application server should not directly execute arbitrary user code with unrestricted permissions.

High-level flow:

```text
User code
 ↓
Backend
 ↓
Isolated code runner
 ↓
Test execution
 ↓
Result
 ↓
Backend
 ↓
Frontend
```

Execution should have limits for:

- CPU
- Memory
- Execution time
- Processes
- Network access
- Filesystem access

---

# 20. User Dashboard

The dashboard should provide useful information such as:

- Overall score
- Questions completed
- Recent submissions
- Assessment history
- Performance information

The exact dashboard features can evolve based on future tasks.

---

# 21. Assessment History

The platform should maintain an assessment history containing relevant information such as:

- Question
- Submission status
- Score change
- Assessment date
- Completion/interruption status

The system should not expose sensitive authentication or OTP information in user-facing history.

---

# 22. Backend Requirements

The backend must control:

- Authentication
- OTP generation
- OTP verification
- Assessment sessions
- Question access
- Submission processing
- Score changes
- Penalties
- Heartbeats
- Interruption detection
- Resume behavior

The frontend should primarily handle presentation and user interaction.

---

# 23. Data Requirements

MongoDB should store application data including:

- Users
- Questions
- Assessment sessions
- Submissions
- Scores
- Relevant assessment events

The database design should support reliable session-state transitions and prevention of duplicate score changes.

---

# 24. Security Requirements

The product must:

- Validate authenticated users
- Authorize assessment sessions server-side
- Protect private test cases
- Protect OTPs
- Rate-limit OTP attempts
- Prevent OTP reuse
- Prevent duplicate scoring
- Prevent duplicate penalties
- Validate user input
- Isolate submitted code
- Keep secrets out of source control
- Avoid trusting frontend score/state information

---

# 25. Non-Goals

The first version does not need to guarantee that a user is physically unable to:

- Switch operating-system applications
- Close the browser
- Shut down the computer
- Kill the browser process

Browser fullscreen alone cannot guarantee these behaviors.

The product goal is to **detect, warn about, and appropriately handle assessment interruptions**, rather than claim perfect OS-level lockdown.

---

# 26. MVP Scope

The initial MVP should focus on:

```text
Authentication
      ↓
Question selection
      ↓
Assessment OTP
      ↓
Fullscreen assessment
      ↓
Code editor
      ↓
Code execution
      ↓
Submission
      ↓
+1 scoring
      ↓
Next question / completed-question quit
      ↓
Unsubmitted exit OTP
      ↓
-2 penalty
      ↓
Unexpected interruption handling
      ↓
-3 penalty
      ↓
Same-question resume with answer reset
      ↓
Score/history
```

---

# 27. Product Success Criteria

The MVP should successfully demonstrate that:

- Students can solve coding questions.
- Interview candidates can practice coding assessments.
- Self-assessment users can test themselves.
- Assessment entry can be protected with email OTP.
- The assessment enters fullscreen.
- Completed questions award +1.
- A completed question can be followed by either the next question or a direct quit.
- Direct quit after completion does not require OTP or incur another penalty.
- Leaving an unsubmitted question requires OTP.
- Successful unsubmitted exit applies -2.
- Unexpected interruption can be detected from backend session state rather than relying solely on frontend closure events.
- Unexpected interruption applies -3 only once.
- Returning users resume the same question with the answer reset.

---

# 28. Product Principle

CodeAira should provide a balance between:

```text
Coding Practice
+
Interview Preparation
+
Self Assessment
+
Controlled Assessment Experience
```

The assessment system should be strict about its defined scoring and session rules while remaining technically realistic about the limitations of browser-based fullscreen and client-side detection.
