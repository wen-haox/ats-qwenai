# Demo Fixtures

Receipts needs stable fixtures so the hackathon demo is reproducible even when external services are slow or rate-limited.

## Fixture Set

| Fixture | Required Behavior |
|---|---|
| Known-good candidate | Resume claims match repository evidence; sandbox execution passes. |
| Weak or forked candidate | Resume overclaims ownership or originality; verifier marks claims unsupported, contradicted, or partially supported. |
| Supported repository | Uses the selected sandbox ecosystem and has deterministic build/test/smoke commands. |
| Unsupported or failing repository | Demonstrates non-disqualifying sandbox failure handling. |
| Job rubric | Scores verified work, authorship, execution result, and role fit. |
| Interview slots | Includes at least two bookable slots and capacity rules. |
| Cached fallback run | Captures one complete agent run for demo resilience. |

## Supported Repository Contract

The first sandbox ecosystem should be one of:

- JavaScript or TypeScript with `package-lock.json` and a deterministic `npm test` or `npm run smoke` command.
- Java with Maven wrapper and a deterministic `./mvnw test` command.

Pick one ecosystem for the MVP and reject the other as `UNSUPPORTED_ECOSYSTEM` until the first path is reliable.

## Known-Good Candidate Shape

- Name and email use synthetic demo data.
- Resume includes at least three concrete claims:
  - Built a named project.
  - Used specific technologies.
  - Implemented or maintained a specific feature.
- GitHub repository contains commits or contribution signals tied to the candidate account.
- Repository is not only a shallow fork.
- Sandbox command passes and emits persisted logs.

## Weak Candidate Shape

- Name and email use synthetic demo data.
- Resume claims primary authorship or advanced skills.
- GitHub repository is a fork, low-originality copy, bulk dump, or has weak authorship signals.
- Claims are classified as `PARTIALLY_SUPPORTED`, `UNSUPPORTED`, or `CONTRADICTED`.
- Dashboard still allows human review and scheduling decisions.

## Cached Fallback Run

The cached fallback should include:

- Candidate submission.
- Extracted claims.
- GitHub evidence payload.
- Authorship/originality result.
- Sandbox execution result.
- Claim verification statuses.
- Credibility profile.
- Rubric decision.
- Human review or override event.
- Scheduling booking.
- Audit log.

