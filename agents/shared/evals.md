# Evals Ledger

Reusable eval task definitions discovered from real QA sessions and known failure patterns.

- **Researcher** reads this at startup to surface known hard cases in RESEARCH.md
- **Optimizer** writes here when QA-REPORT.md contains `## Eval Candidates` sections
- **Format**: task definition + grader type + pass metric + saturation count

Grader types: **code-based** (deterministic — unit tests, assertions, binary checks) |
**model-based** (rubric scoring, tone/quality, LLM judge) | **human** (expert review)

Pass metrics: **pass@k** (at least one success across k attempts is acceptable) |
**pass^k** (must succeed on every attempt — no flakiness tolerated)

---

## AWS / Lambda

### DynamoDB Reserved Word in Query Expression
- **Task**: Write a DynamoDB query/scan that filters on `status`, `name`, `type`, or other reserved words
- **Input**: Lambda handler using FilterExpression with a reserved word field
- **Success criteria**: Query executes without `ValidationException`; uses `ExpressionAttributeNames` aliases (e.g. `#status`)
- **Grader type**: code-based
- **pass metric**: pass^k
- **Reference solution**: any existing handler using `#status` alias
- **Source**: Known pattern, documented in solutions.md
- **Added**: 2026-03-26
- **saturation_count**: 0

---

## Backend / Payouts

### Payout Idempotency Key Prevents Duplicate Payout
- **Task**: Payout processor Lambda calls an external payments API to send payment
- **Input**: Retry scenario — same user ID + period called twice
- **Success criteria**: Second call does not create a second transaction; external API receives the same `idempotencyKey` and deduplicates
- **Grader type**: code-based
- **pass metric**: pass^k
- **Reference solution**: Payout Lambda with idempotencyKey pattern `payout-{userId}-{periodStart}`
- **Source**: Known pattern — duplicate payout risk on Lambda retry
- **Added**: 2026-03-26
- **saturation_count**: 0

---

## Mobile / IAP

### Google Play purchaseState Must Be Exactly 0
- **Task**: IAP verification Lambda validates a Google Play purchase token
- **Input**: Receipt with `purchaseState` = 0 (valid), 1 (canceled), 2 (pending)
- **Success criteria**: `purchaseState=0` → purchase accepted; `purchaseState=1` or `2` → rejected with 400
- **Grader type**: code-based
- **pass metric**: pass^k
- **Reference solution**: IAP verification Lambda — Google Play handler
- **Source**: Known pattern — values 1/2 are easy to miss
- **Added**: 2026-03-26
- **saturation_count**: 0
