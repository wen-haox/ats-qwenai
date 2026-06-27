# Product Requirements Document

**Project:** Receipts — Proof-of-Execution Hiring Agent  
**Hackathon:** Qwen Cloud Global AI Hackathon  
**Track:** Track 4 — Autopilot Agent  
**Submission Deadline:** July 9, 2026

---

## 1. Summary

Receipts is an autonomous hiring verification agent that evaluates a candidate's claimed work using concrete, evidence-linked receipts instead of relying on resumes alone.

Given a candidate's resume, GitHub repositories, live demos, writeups, and a job-specific evaluation rubric, the system harvests evidence, verifies authorship and originality, executes selected candidate code in an isolated sandbox, cross-checks claims against deterministic evidence, generates a credibility profile, scores the candidate against the rubric, and supports interview scheduling with human review and override.

The core product invariant is:

> A claim is not marked as verified because a model says so. A claim is marked as verified only when deterministic code confirms that a concrete evidence pointer exists and supports the claim.

Receipts is designed as a human-controlled agentic workflow: every automated decision is explainable, overridable, and audit-logged.

---

## 2. Problem Statement

Modern recruiting workflows are increasingly affected by AI-polished resumes, copied project descriptions, forked repositories presented as original work, and claims that do not match actual contribution history.

Traditional applicant tracking systems primarily perform keyword matching and resume ranking. They do not reliably verify whether a candidate actually authored the claimed work, whether a repository is original or mostly forked, whether the code runs, or whether a stated skill is supported by concrete evidence.

Hiring teams need an evidence-first workflow that distinguishes genuine demonstrated competence from polished self-presentation while keeping humans in control of final decisions.

---

## 3. Goals

### 3.1 Product Goals

- Verify candidate claims against real artifacts such as repositories, commits, files, test results, demo pages, and writeups.
- Produce a credibility profile where each claim is linked to evidence and assigned a verification status.
- Score candidates against a configurable job rubric based on evidence-backed signals.
- Detect repository authorship, originality, forks, and suspicious contribution patterns.
- Execute candidate code in an isolated sandbox for proof-of-execution.
- Provide a dashboard for hiring teams to review decisions, inspect traces, override outcomes, and audit actions.
- Support interview scheduling through magic-link slot booking.
- Demonstrate an end-to-end agentic workflow powered by Qwen Cloud, external tools, custom skills, and human checkpoints.

### 3.2 Technical Goals

- Use Qwen Cloud models for planning, extraction, classification, scoring, and structured outputs.
- Use a tool-calling agent loop with custom verification tools.
- Use deterministic verification code to validate model-proposed evidence.
- Store all agent runs, tool calls, evidence, decisions, and audit events.
- Deploy the backend and supporting services on Alibaba Cloud infrastructure.
- Provide a stable demo with fixture candidates, fixture repositories, and cached fallback results.

---

## 4. Non-Goals

The following are out of scope for the hackathon version:

- Full applicant tracking system or recruiting CRM.
- Multi-tenant enterprise workspace management.
- Enterprise SSO, RBAC beyond a minimal admin/operator distinction, or organization-wide account management.
- Fine-tuning or training custom models.
- Full free-text email availability parsing.
- Full calendar integration if magic-link scheduling is sufficient for demo.
- Multi-language or arbitrary-ecosystem sandbox support.
- Fully automated candidate rejection without human visibility and override.
- Production-grade plagiarism or AI-generated-code detection.
- Mobile applications.

---

## 5. Users and Personas

| Persona | Role | Primary Needs |
|---|---|---|
| Hiring Manager / Recruiter | Primary operator | Configure jobs and rubrics, review credibility profiles, inspect evidence, override decisions, manage interview slots. |
| Candidate | Subject of verification and scheduling | Submit resume and work links, receive scheduling link if advanced, optionally view a transparent profile of verified work. |
| Admin | Setup operator | Configure environment, integrations, API keys, deployment, and system-level settings. |

---

## 6. Core Product Principles

### 6.1 Evidence-First Verification

Every claim must be linked to concrete evidence before it can be considered verified. Evidence may include:

- Commit SHA
- Repository URL
- File path
- Pull request URL
- Test-run ID
- Sandbox execution log
- Demo URL
- Writeup URL
- Static-analysis result
- Authorship analysis result
- Fork/originality analysis result

### 6.2 Deterministic Receipts Invariant

The LLM may propose claims, reasoning, and candidate evidence pointers. Deterministic code must validate that the evidence exists and supports the claim before the system marks the claim as `VERIFIED`.

### 6.3 Human Control

The system may recommend outcomes, but hiring users must be able to inspect, override, and audit every automated decision.

### 6.4 Non-Disqualifying Sandbox Outcomes

Sandbox execution failures must not automatically reject a candidate. Execution results affect claim confidence and candidate evidence strength, but final decisions must remain reviewable and overridable.

### 6.5 Narrow Demo Scope

The hackathon version must prioritize a reliable vertical slice over broad feature coverage. The winning demo path is evidence harvesting, authorship/originality analysis, sandbox execution, claim verification, fit decision, human override, and scheduling link.

---

## 7. Functional Requirements

Priorities:

- **P0:** Required for hackathon demo.
- **P1:** Strongly desired if time allows.
- **P2:** Stretch.

---

## 8. Module A — Job, Rubric, and Intake

| ID | Requirement | Priority |
|---|---|---|
| FR-A1 | Hiring user can create a job posting with title, description, and required skills. | P0 |
| FR-A2 | Hiring user can define a weighted evaluation rubric for the job. | P0 |
| FR-A3 | Hiring user can configure interview slots with date, time, timezone, and capacity. | P0 |
| FR-A4 | Candidate can submit resume text or a resume file plus work links. | P0 |
| FR-A5 | Candidate can submit GitHub repository URLs. | P0 |
| FR-A6 | Candidate can submit live demo and writeup URLs. | P1 |
| FR-A7 | System validates submitted links for reachability and supported type. | P1 |
| FR-A8 | System stores submitted artifacts and associates them with the candidate. | P0 |

---

## 9. Module B — Agent Run Lifecycle

| ID | Requirement | Priority |
|---|---|---|
| FR-B1 | Submitting a candidate triggers an `AgentRun`. | P0 |
| FR-B2 | An `AgentRun` has a persisted lifecycle state. | P0 |
| FR-B3 | Agent progress is persisted through `ToolCall`, `Evidence`, `Decision`, and `AuditEvent` records. | P0 |
| FR-B4 | Agent runs are idempotent and resumable from persisted tool-call history. | P0 |
| FR-B5 | Tool failures are classified as retryable or terminal. | P0 |
| FR-B6 | Transient failures retry with backoff. | P1 |
| FR-B7 | Dashboard can display the current run state. | P0 |
| FR-B8 | Dashboard can stream or poll the agent trace. | P1 |

### 9.1 AgentRun States

```text
QUEUED
RUNNING
WAITING_FOR_TOOL
COMPLETED
FAILED_RETRYABLE
FAILED_TERMINAL
CANCELLED
```

---

## 10. Module C — Claim Extraction

| ID | Requirement | Priority |
|---|---|---|
| FR-C1 | Agent extracts structured claims from resume text. | P0 |
| FR-C2 | Claims include skills, projects, roles, impact statements, and technology usage. | P0 |
| FR-C3 | Claims are initially stored as `UNVERIFIED`. | P0 |
| FR-C4 | Claims include source text spans where possible. | P1 |
| FR-C5 | Claims are normalized for matching against repository evidence and rubric criteria. | P0 |

### 10.1 Claim Statuses

```text
UNVERIFIED
VERIFIED
PARTIALLY_SUPPORTED
UNSUPPORTED
CONTRADICTED
```

---

## 11. Module D — Evidence Harvesting

| ID | Requirement | Priority |
|---|---|---|
| FR-D1 | Agent fetches GitHub repository metadata. | P0 |
| FR-D2 | Agent fetches commit history. | P0 |
| FR-D3 | Agent fetches contributors and authorship-related signals. | P0 |
| FR-D4 | Agent fetches repository language and file-structure information. | P0 |
| FR-D5 | Agent fetches pull request and issue activity if available. | P1 |
| FR-D6 | Agent fetches live demo pages and writeups. | P1 |
| FR-D7 | Harvested artifacts are persisted and linkable as evidence. | P0 |
| FR-D8 | Evidence records include concrete pointers such as commit SHA, file path, URL, test-run ID, or log object key. | P0 |

---

## 12. Module E — Forensic Verification Engine

The forensic verification engine is the core product differentiator.

| ID | Requirement | Priority |
|---|---|---|
| FR-E1 | System performs authorship analysis using GitHub-account-tied signals where available. | P0 |
| FR-E2 | System avoids relying only on spoofable local git config author names or emails. | P0 |
| FR-E3 | System calculates candidate commit share per repository. | P0 |
| FR-E4 | System calculates candidate changed-line share where feasible. | P1 |
| FR-E5 | System classifies candidate role as primary author, meaningful contributor, minor contributor, or unclear. | P0 |
| FR-E6 | System detects whether a repository is a fork. | P0 |
| FR-E7 | System compares forked repositories against upstream where possible. | P0 |
| FR-E8 | System estimates candidate-original contribution in forked repositories. | P0 |
| FR-E9 | System flags suspicious contribution patterns such as bulk dumps or extremely compressed commit history. | P1 |
| FR-E10 | System performs basic static analysis for supported ecosystems. | P1 |
| FR-E11 | System detects unsupported or insufficient evidence and marks affected claims accordingly. | P0 |

---

## 13. Module F — Sandbox Proof-of-Execution

Sandbox execution is required for the hackathon demo and must support one ecosystem only.

| ID | Requirement | Priority |
|---|---|---|
| FR-F1 | System can clone a selected candidate repository into an ephemeral work directory. | P0 |
| FR-F2 | System detects whether the repository matches the supported sandbox ecosystem. | P0 |
| FR-F3 | System uses a pre-baked image for the supported ecosystem. | P0 |
| FR-F4 | System prepares dependencies in a controlled dependency-preparation phase. | P0 |
| FR-F5 | System runs proof execution in a network-disabled phase. | P0 |
| FR-F6 | System runs candidate code as a non-root user. | P0 |
| FR-F7 | System uses a read-only root filesystem where feasible. | P0 |
| FR-F8 | System uses an ephemeral writable work directory or tmpfs. | P0 |
| FR-F9 | System enforces CPU, memory, PID, disk, and wall-clock limits. | P0 |
| FR-F10 | System captures build/test/smoke-test command output. | P0 |
| FR-F11 | System records execution status, logs, duration, and failure reason. | P0 |
| FR-F12 | System stores sandbox logs and artifacts in object storage. | P0 |
| FR-F13 | System destroys sandbox containers after each run and never reuses them. | P0 |
| FR-F14 | System treats unsupported ecosystems as non-disqualifying. | P0 |

### 13.1 Sandbox Phases

#### Phase A — Dependency Preparation

- Network may be enabled only for controlled dependency preparation.
- Dependency fetch must have strict timeout and size limits.
- Package registry access should be allowlisted where feasible.
- The output is a prepared dependency cache, lockfile snapshot, or build environment for the execution phase.

#### Phase B — Proof Execution

- Network is disabled.
- Code runs as a non-root user.
- Root filesystem is read-only where feasible.
- Writable area is temporary and isolated.
- CPU, memory, PID, disk, and wall-clock limits are enforced.
- Build, test, or smoke-test commands are executed.
- Logs and outcomes are captured and persisted.

### 13.2 Sandbox Execution Statuses

```text
EXECUTION_PASSED
EXECUTION_FAILED
UNSUPPORTED_ECOSYSTEM
DEPENDENCY_FETCH_FAILED
TIMEOUT
SECURITY_BLOCKED
NO_TESTS_FOUND
INTERNAL_ERROR
```

---

## 14. Module G — Claim-to-Evidence Cross-Check

| ID | Requirement | Priority |
|---|---|---|
| FR-G1 | Agent proposes evidence pointers for each extracted claim. | P0 |
| FR-G2 | Deterministic code validates that each proposed evidence pointer exists. | P0 |
| FR-G3 | Deterministic code validates that the evidence type is eligible to support the claim. | P0 |
| FR-G4 | Claims are marked `VERIFIED` only when validated evidence supports them. | P0 |
| FR-G5 | Claims are marked `PARTIALLY_SUPPORTED` when evidence supports only part of the claim. | P0 |
| FR-G6 | Claims are marked `UNSUPPORTED` when no adequate evidence is found. | P0 |
| FR-G7 | Claims are marked `CONTRADICTED` when evidence conflicts with the claim. | P0 |
| FR-G8 | Every non-`UNVERIFIED` claim status must include one or more evidence links. | P0 |
| FR-G9 | The system stores model reasoning separately from deterministic verification results. | P0 |

---

## 15. Module H — Credibility Profile

| ID | Requirement | Priority |
|---|---|---|
| FR-H1 | System generates a credibility profile for each processed candidate. | P0 |
| FR-H2 | Profile lists extracted claims and verification statuses. | P0 |
| FR-H3 | Profile links each verified or contradicted claim to concrete evidence. | P0 |
| FR-H4 | Profile includes authorship and originality summaries for submitted repositories. | P0 |
| FR-H5 | Profile includes sandbox execution results where available. | P0 |
| FR-H6 | Profile includes confidence levels for major findings. | P0 |
| FR-H7 | Profile includes a human-readable explanation of the decision. | P0 |
| FR-H8 | Profile can be viewed from the hiring dashboard. | P0 |
| FR-H9 | Profile can optionally be shared through a public or magic-link URL. | P1 |

---

## 16. Module I — Fit Evaluation and Decisioning

| ID | Requirement | Priority |
|---|---|---|
| FR-I1 | System scores the candidate against the configured job rubric. | P0 |
| FR-I2 | Scoring uses evidence-backed signals and verified claims. | P0 |
| FR-I3 | Scoring does not use demographic, protected-class, or pedigree-only signals. | P0 |
| FR-I4 | System produces one of the supported decision outcomes. | P0 |
| FR-I5 | Decision includes a rationale linked to evidence and rubric criteria. | P0 |
| FR-I6 | Decision includes uncertainty or missing-evidence notes. | P1 |
| FR-I7 | Decision is persisted and audit-logged. | P0 |

### 16.1 Decision Outcomes

```text
INTERVIEW
BORDERLINE
NO
NEEDS_HUMAN_REVIEW
```

---

## 17. Module J — Human Control Plane and Audit

| ID | Requirement | Priority |
|---|---|---|
| FR-J1 | Dashboard shows all candidates for a job and their current stage. | P0 |
| FR-J2 | Dashboard shows candidate credibility profile and decision. | P0 |
| FR-J3 | Dashboard shows evidence links and tool-call trace. | P0 |
| FR-J4 | Hiring user can override any automated decision. | P0 |
| FR-J5 | Override requires a reason or note. | P1 |
| FR-J6 | Override updates candidate stage. | P0 |
| FR-J7 | Override is recorded in the audit log with actor, timestamp, before state, after state, and reason. | P0 |
| FR-J8 | Every automated action is recorded as an audit event. | P0 |
| FR-J9 | Audit log is append-only at the application level. | P0 |

---

## 18. Module K — Scheduling

| ID | Requirement | Priority |
|---|---|---|
| FR-K1 | Recommended candidates receive or are assigned a magic-link scheduling URL. | P0 |
| FR-K2 | Scheduling URL is visible in the dashboard for demo reliability. | P0 |
| FR-K3 | Candidate can open the scheduling link and view available interview slots. | P0 |
| FR-K4 | Candidate can book a slot atomically. | P0 |
| FR-K5 | System prevents double-booking beyond slot capacity. | P0 |
| FR-K6 | Booking creates an `Interview` record. | P0 |
| FR-K7 | System can send the scheduling link by email. | P1 |
| FR-K8 | System can send confirmation emails or calendar invites. | P2 |

---

## 19. Module L — Dashboard and Experience

| ID | Requirement | Priority |
|---|---|---|
| FR-L1 | Hiring dashboard supports job and rubric configuration. | P0 |
| FR-L2 | Dashboard supports candidate intake or candidate submission review. | P0 |
| FR-L3 | Dashboard shows pipeline stages. | P0 |
| FR-L4 | Dashboard shows agent run status. | P0 |
| FR-L5 | Dashboard shows reasoning trace through SSE or polling. | P1 |
| FR-L6 | Dashboard shows credibility profile with claim statuses and evidence links. | P0 |
| FR-L7 | Dashboard supports human override. | P0 |
| FR-L8 | Dashboard surfaces scheduling links for recommended candidates. | P0 |
| FR-L9 | Dashboard surfaces audit events. | P0 |

---

## 20. Module M — Deployment and Submission Readiness

| ID | Requirement | Priority |
|---|---|---|
| FR-M1 | Backend is deployed on Alibaba Cloud infrastructure. | P0 |
| FR-M2 | Deployment region is `ap-southeast-1` where feasible. | P0 |
| FR-M3 | Project repository includes setup and deployment instructions. | P0 |
| FR-M4 | Repository includes an architecture diagram. | P0 |
| FR-M5 | Repository includes proof or documentation of Alibaba Cloud deployment. | P0 |
| FR-M6 | Repository identifies the hackathon track. | P0 |
| FR-M7 | Submission includes a public demo video. | P0 |
| FR-M8 | Submission includes a concise product description and technical summary. | P0 |

---

## 21. Data Model Requirements

### 21.1 Core Entities

The system must include the following core entities:

- `JobPosting`
- `Rubric`
- `RubricCriterion`
- `InterviewSlot`
- `Candidate`
- `Artifact`
- `Claim`
- `Evidence`
- `VerificationResult`
- `CredibilityProfile`
- `Decision`
- `Interview`
- `AgentRun`
- `ToolCall`
- `AuditEvent`

### 21.2 Entity Requirements

#### JobPosting

Stores job configuration.

Required fields:

- ID
- Title
- Description
- Required skills
- Created timestamp
- Updated timestamp

#### Rubric

Stores evaluation criteria for a job.

Required fields:

- ID
- Job posting ID
- Criteria
- Weights
- Created timestamp
- Updated timestamp

#### Candidate

Stores candidate information and submission state.

Required fields:

- ID
- Job posting ID
- Name
- Email
- Resume text or resume artifact reference
- Current stage
- Created timestamp
- Updated timestamp

#### Artifact

Stores submitted and harvested artifacts.

Required fields:

- ID
- Candidate ID
- Artifact type
- Source URL or object key
- Normalized reference
- Created timestamp

Supported artifact types:

```text
RESUME
GITHUB_REPO
LIVE_DEMO
WRITEUP
SANDBOX_LOG
STATIC_ANALYSIS_RESULT
OTHER
```

#### Claim

Stores extracted candidate claims.

Required fields:

- ID
- Candidate ID
- Claim text
- Claim type
- Source artifact ID
- Status
- Confidence
- Created timestamp
- Updated timestamp

Supported claim types:

```text
SKILL
PROJECT
ROLE
IMPACT
TECHNOLOGY
CONTRIBUTION
OTHER
```

#### Evidence

Stores concrete evidence pointers.

Required fields:

- ID
- Candidate ID
- Artifact ID
- Evidence type
- Pointer
- Summary
- Created timestamp

Supported evidence types:

```text
COMMIT_SHA
FILE_PATH
PULL_REQUEST_URL
REPOSITORY_URL
TEST_RUN_ID
SANDBOX_LOG
DEMO_URL
WRITEUP_URL
AUTHORSHIP_RESULT
ORIGINALITY_RESULT
STATIC_ANALYSIS_RESULT
```

#### VerificationResult

Stores forensic verification output for an artifact.

Required fields:

- ID
- Artifact ID
- Authorship summary
- Originality summary
- Execution summary
- Status
- Created timestamp

#### CredibilityProfile

Stores the assembled profile.

Required fields:

- ID
- Candidate ID
- Summary
- Verified claims
- Unsupported claims
- Contradicted claims
- Evidence summary
- Created timestamp
- Updated timestamp

#### Decision

Stores the agent recommendation.

Required fields:

- ID
- Candidate ID
- Outcome
- Rubric score
- Rationale
- Evidence summary
- Created timestamp
- Updated timestamp

#### InterviewSlot

Stores available interview slots.

Required fields:

- ID
- Job posting ID
- Start time
- End time
- Timezone
- Capacity
- Booked count

#### Interview

Stores booked interviews.

Required fields:

- ID
- Candidate ID
- Interview slot ID
- Status
- Created timestamp
- Updated timestamp

#### AgentRun

Stores an agent execution instance.

Required fields:

- ID
- Candidate ID
- Status
- Started timestamp
- Completed timestamp
- Error summary
- Created timestamp
- Updated timestamp

#### ToolCall

Stores tool invocation traces.

Required fields:

- ID
- Agent run ID
- Tool name
- Status
- Input
- Output
- Evidence IDs
- Model used
- Started timestamp
- Completed timestamp
- Error

#### AuditEvent

Stores append-only audit records.

Required fields:

- ID
- Actor type
- Actor ID
- Candidate ID
- Action
- Before state
- After state
- Reason
- Created timestamp

---

## 22. API Requirements

The platform must expose REST APIs and either SSE or polling endpoints for run progress.

### 22.1 Required REST Resources

```text
/jobs
/jobs/{jobId}/rubric
/jobs/{jobId}/slots
/jobs/{jobId}/candidates
/candidates/{candidateId}
/candidates/{candidateId}/artifacts
/candidates/{candidateId}/agent-runs
/candidates/{candidateId}/profile
/candidates/{candidateId}/decision
/candidates/{candidateId}/override
/candidates/{candidateId}/audit-events
/scheduling/{magicToken}
/scheduling/{magicToken}/book
```

### 22.2 Trace Streaming

Preferred:

```text
GET /agent-runs/{agentRunId}/events
```

Fallback:

```text
GET /agent-runs/{agentRunId}
GET /agent-runs/{agentRunId}/tool-calls
```

---

## 23. Tool Interface Requirements

Each tool must have a typed input, typed output, timeout, retry policy, and deterministic status.

### 23.1 Tool Result Envelope

```json
{
  "toolCallId": "uuid",
  "toolName": "github.fetchRepository",
  "status": "SUCCESS",
  "input": {},
  "output": {},
  "evidenceIds": [],
  "startedAt": "timestamp",
  "endedAt": "timestamp",
  "error": null
}
```

### 23.2 Tool Statuses

```text
SUCCESS
FAILED_RETRYABLE
FAILED_TERMINAL
SKIPPED
TIMEOUT
```

### 23.3 Required Tools

| Tool | Responsibility | Priority |
|---|---|---|
| GitHub tool | Fetch repository metadata, commits, contributors, forks, and file structure. | P0 |
| Authorship tool | Calculate candidate authorship and contribution signals. | P0 |
| Originality tool | Detect forks and estimate original contribution. | P0 |
| Sandbox runner | Build, test, or smoke-test a supported repository. | P0 |
| Claim verifier | Validate model-proposed evidence pointers. | P0 |
| Rubric scorer | Score verified profile against job rubric. | P0 |
| Scheduler | Generate magic links and book slots. | P0 |
| Web fetch tool | Fetch demo pages and writeups. | P1 |
| Static analysis tool | Produce basic code structure and complexity signals. | P1 |
| Email tool | Send scheduling links and confirmations. | P1 |

---

## 24. Agent Requirements

| ID | Requirement | Priority |
|---|---|---|
| AR-1 | Agent uses Qwen Cloud for planning, extraction, classification, and scoring. | P0 |
| AR-2 | Agent uses a larger model for planning and judgment. | P0 |
| AR-3 | Agent uses a cheaper/faster model for extraction and classification where appropriate. | P1 |
| AR-4 | Agent uses structured outputs for claims and decisions. | P0 |
| AR-5 | Agent invokes tools through a defined interface. | P0 |
| AR-6 | Agent persists each model step and tool call. | P0 |
| AR-7 | Agent separates model reasoning from deterministic verification state. | P0 |
| AR-8 | Agent stops when sufficient evidence has been collected or when a configured budget is reached. | P0 |
| AR-9 | Agent handles missing evidence by lowering confidence or marking claims unsupported rather than fabricating conclusions. | P0 |

---

## 25. Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Security | Untrusted code must run only inside the sandbox. |
| NFR-2 | Sandbox isolation | Proof execution must run network-disabled, non-root, resource-limited, and single-use. |
| NFR-3 | Reliability | Agent runs must be idempotent and resumable where feasible. |
| NFR-4 | Observability | Every agent step, tool call, decision, and override must be logged and persisted. |
| NFR-5 | Auditability | Automated and human actions must be recorded in append-only audit events. |
| NFR-6 | Performance | Verification should use concurrent I/O fan-out where safe. |
| NFR-7 | Sandbox concurrency | Sandbox execution must use a bounded pool or semaphore separate from general I/O concurrency. |
| NFR-8 | Cost | Model routing and prompt reuse should keep usage within hackathon voucher limits. |
| NFR-9 | Privacy | Decisions must avoid demographic, protected-class, or pedigree-only signals. |
| NFR-10 | Deployment | Backend and required services must run on Alibaba Cloud infrastructure for submission. |
| NFR-11 | Demo reliability | Demo must support pre-staged fixture candidates and fallback cached results. |
| NFR-12 | Maintainability | Tool interfaces and data contracts must be explicit and stable after initial implementation. |

---

## 26. Concurrency and Orchestration Requirements

| ID | Requirement | Priority |
|---|---|---|
| OR-1 | Agent runs are queued before execution. | P0 |
| OR-2 | I/O-bound verification tasks may run concurrently using virtual threads or equivalent. | P0 |
| OR-3 | Sandbox tasks must be isolated from general I/O concurrency through a bounded executor, pool, or semaphore. | P0 |
| OR-4 | Duplicate candidate submissions or retries must not create inconsistent duplicate decisions. | P0 |
| OR-5 | Tool calls must be safe to retry when marked retryable. | P1 |
| OR-6 | Run state must be recoverable after process restart where feasible. | P1 |

---

## 27. Security Requirements

| ID | Requirement | Priority |
|---|---|---|
| SEC-1 | API keys and secrets must be environment-injected and never committed. | P0 |
| SEC-2 | GitHub tokens must be stored securely and scoped minimally. | P0 |
| SEC-3 | Candidate code must never execute in the backend process. | P0 |
| SEC-4 | Sandbox containers must be single-use. | P0 |
| SEC-5 | Sandbox logs must not expose system secrets. | P0 |
| SEC-6 | Magic links must be signed and time-limited. | P0 |
| SEC-7 | Slot booking must be atomic to prevent double-booking. | P0 |
| SEC-8 | Human overrides must be audit-logged. | P0 |

---

## 28. Demo Requirements

The demo must show the following end-to-end flow:

1. Hiring user configures a job, rubric, and interview slots.
2. Candidate submits resume text and at least one GitHub repository.
3. Agent extracts claims from the candidate submission.
4. Agent harvests GitHub evidence.
5. Agent identifies authorship and originality signals.
6. Agent runs proof-of-execution in the sandbox for one supported repository.
7. Agent marks claims as `VERIFIED`, `PARTIALLY_SUPPORTED`, `UNSUPPORTED`, or `CONTRADICTED`.
8. Dashboard displays an evidence-linked credibility profile.
9. Agent scores candidate against the rubric and emits a decision.
10. Hiring user reviews trace and evidence.
11. Hiring user overrides or confirms the decision.
12. Candidate scheduling link is created and visible in the dashboard.
13. Candidate books an interview slot.
14. Audit log shows the automated and human actions.
15. Demo briefly shows Alibaba Cloud deployment proof.

---

## 29. Required Demo Fixtures

The project must include or prepare the following fixtures:

| Fixture | Purpose |
|---|---|
| Known-good candidate | Shows verified claims and successful sandbox execution. |
| Known-fake or weak candidate | Shows fork/originality detection and unsupported or contradicted claims. |
| Supported-ecosystem repository | Used for stable sandbox execution. |
| Failing or unsupported repository | Demonstrates non-disqualifying failure handling. |
| Job rubric | Used for repeatable scoring. |
| Interview slots | Used for scheduling demo. |
| Cached fallback run | Used if live external services are slow or unavailable. |

---

## 30. Cut Lines

If implementation falls behind, features must be cut in the following order:

1. AI-generated-code or plagiarism detection.
2. Calendar invite generation.
3. Free-text writeup reading.
4. Public candidate-facing profile.
5. Email deliverability.
6. Live SSE trace, replaced by polling.
7. Static analysis beyond minimal repository inspection.
8. Multi-candidate batch processing.
9. pgvector retrieval if deterministic claim-evidence matching is sufficient.
10. Full resume PDF parsing, replaced by text upload.

The following must not be cut:

- GitHub evidence harvesting.
- Authorship analysis.
- Fork/originality detection.
- Sandbox proof-of-execution for one ecosystem.
- Evidence-linked claim verification.
- Fit decision against rubric.
- Human override.
- Audit log.
- Magic-link scheduling flow or dashboard-visible scheduling link.
- Alibaba Cloud deployment proof.

---

## 31. Milestones

### Day 1 — Integration Spike

Required output:

- Qwen Cloud model call works.
- One tool-call round trip works.
- One GitHub tool call works.
- Structured output is persisted.
- Initial repository scaffold exists.

### Day 2 — Contract Freeze

Required output:

- Data entities defined.
- REST API shapes defined.
- Tool result envelope defined.
- AgentRun lifecycle defined.
- Fixture candidates and fixture repositories selected.

### Days 3–4 — Sandbox MVP

Required output:

- One supported ecosystem selected.
- Dependency-preparation phase works.
- Network-disabled execution phase works.
- Resource limits are enforced.
- Logs and statuses are persisted.
- Known-good and failing fixture repos are tested.

### Days 5–6 — Verification Engine

Required output:

- Claim extraction works.
- GitHub harvesting works.
- Authorship analysis works.
- Fork/originality detection works.
- Claim-to-evidence cross-check works.

### Days 7–8 — Platform Experience

Required output:

- Dashboard displays candidates and stages.
- Credibility profile view works.
- Evidence links are visible.
- Decision view works.
- Override and audit work.
- Scheduling link flow works.

### Day 9 — End-to-End Vertical Slice

Required output:

- One candidate can be processed from intake to decision.
- Dashboard shows profile, evidence, decision, override, audit, and scheduling.

### Day 10 — Alibaba Cloud Deployment

Required output:

- Backend is deployed on Alibaba Cloud.
- Database and object storage are configured.
- Deployment proof is documented.
- Environment variables and secrets are configured safely.

### Day 11 — Demo Hardening

Required output:

- Demo script is finalized.
- Fixture data is stable.
- Cached fallback results are prepared.
- Failure paths are handled gracefully.

### Day 12 — Submission

Required output:

- Public repository is complete.
- README includes setup, architecture, deployment, and demo instructions.
- Architecture diagram is included.
- Demo video is recorded.
- Devpost submission text is finalized.
- Track is clearly identified.

---

## 32. Success Metrics

| Metric | Target |
|---|---|
| End-to-end demo completion | Candidate intake to decision to scheduling works. |
| Claim verification | At least three claims are extracted and classified with evidence. |
| Sandbox proof | At least one repository produces a persisted sandbox execution result. |
| Authorship detection | At least one repository includes candidate authorship analysis. |
| Fork/originality detection | At least one fixture demonstrates fork or low-originality detection. |
| Human override | At least one decision override is recorded in audit log. |
| Alibaba deployment | Backend deployment proof is available. |
| Demo reliability | Cached fallback exists for slow or failing external services. |

---

## 33. Submission Requirements

The final submission must include:

- Public source repository.
- `README.md` with setup and run instructions.
- `prd.md`.
- Architecture diagram.
- Demo video.
- Track identification: Track 4 — Autopilot Agent.
- Description of Qwen Cloud usage.
- Description of Alibaba Cloud deployment.
- Proof of deployed backend on Alibaba Cloud.
- License file.
- Demo fixtures or instructions for reproducing the demo.
