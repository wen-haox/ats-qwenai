# Demo Script

Target length: under three minutes. The script should privilege the end-to-end story over implementation detail, then use the repository and architecture docs for technical depth.

## Opening

Receipts is a proof-of-execution hiring agent for Track 4 — Autopilot Agent. It addresses AI-polished resumes and copied project claims by verifying candidate statements against concrete evidence: GitHub activity, repository originality, sandbox execution, and audit-logged human review.

## Flow

1. Show the configured job, weighted rubric, and interview slots.
2. Submit the known-good candidate fixture with resume text and a GitHub repository.
3. Start the agent run.
4. Show Qwen extracting structured claims.
5. Show GitHub evidence harvesting and authorship/originality results.
6. Show the sandbox result for the supported repository.
7. Show deterministic claim verification with evidence links.
8. Show the credibility profile and rubric-backed recommendation.
9. Confirm or override the recommendation as the hiring user.
10. Show the audit log entry for automated and human actions.
11. Open the dashboard-visible scheduling link and book one slot.
12. Show Alibaba Cloud deployment proof.

## Lines To Emphasize

- Qwen plans and reasons; deterministic tools verify evidence.
- The system produces recommendations, not automatic final hiring decisions.
- Every important action has a trace and audit event.
- Unsupported or failed sandbox execution is non-disqualifying and visible to the reviewer.
- The MVP is narrow by design: one reliable vertical slice with fixture-backed fallback.

## Backup Path

If a live external service is slow, use the cached fallback run and call that out in the video. The fallback must still show Qwen outputs, tool results, evidence links, sandbox status, decision, review, scheduling, and audit entries.

