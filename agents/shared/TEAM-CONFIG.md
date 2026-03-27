# Team Shared Config

Global standards for the development team. Every agent reads this file on startup, then reads the project's `CLAUDE.md` for overrides. **Project rules always win.**

## Protocol

1. Read this file on startup
2. Read `~/.claude/agents/shared/solutions.md` — **skip any approach listed as a failed approach before trying it**
3. Read the project's `CLAUDE.md` (overrides these globals)
4. Load only the referenced files relevant to your task — don't load everything
5. Write all task output to the workspace directory provided by the orchestrator
6. Never re-ask for context already in these files

## Error Reporting (Required)

When you encounter an error, try a different approach, and succeed — document it. At the end of your workspace output file, add:

```markdown
## Solutions Found

### <Short Title>
- **Error**: <exact error message or symptom>
- **Failed approach**: <what was tried first>
- **Solution**: <what actually worked — be specific>
- **Scope**: <global | agent-name | tool-name>
```

- Add one entry per distinct error→solution pair discovered during the task
- **Always include this section** — write `None found.` if no errors required a different approach. Never omit the section header.
- The Optimizer reads these and propagates solutions to the shared ledger automatically
- **Do not** document expected/intentional failures (e.g. a 404 you were checking for)

## Efficiency Reporting (Required)

When you notice you wasted steps — loaded a file you never used, ran the same search twice, read context irrelevant to the task — document it. At the end of your workspace output file, add:

```markdown
## Efficiency Signals

### <Short Title>
- **Waste observed**: <what step or tool call was unnecessary>
- **Why it was wasteful**: <loaded X but never used it / ran grep twice / etc.>
- **Suggested fix**: <skip this step when Y / merge steps A and B / conditionally load only if Z>
- **Scope**: <global | agent-name>
```

- Add one entry per distinct waste pattern noticed
- **Always include this section** — write `None observed.` if no waste was noticed. Never omit the section header.
- Be honest — self-reporting inefficiency is how the whole team gets faster
- The Optimizer reads these and applies targeted edits to agent files when a pattern is confirmed

## Primary Stack

| Layer | Technology |
|---|---|
| Language (primary) | TypeScript (strict mode) |
| Language (device) | Python 3.10+ |
| Mobile | React Native + Expo (local builds only) |
| Backend | AWS Lambda (Node.js 20.x) |
| API | AWS API Gateway (REST) |
| Database | DynamoDB |
| Infrastructure | AWS CDK (TypeScript) |
| CI/CD | GitHub Actions |
| Formatting | Prettier (enforced pre-commit) |
| Region | us-west-2 (default) |

## Task Workspace Convention

The orchestrator creates a workspace at `.agent-team/<date>-<slug>/` for each task.

| File | Written by | Contains |
|---|---|---|
| `TASK.md` | Orchestrator | Task description, scope, agent assignments |
| `RESEARCH.md` | Researcher | Findings, references, patterns investigated |
| `BRIEF.md` | Prompt Coach | Sharpened brief — confirmed tool choices, testable ACs, resolved decisions. Primary input for Architect when present. |
| `ARCHITECTURE.md` | Architect | Design decisions, API contracts, data model, component breakdown |
| `UI-SPEC.md` | Designer | User flows, wireframes (ASCII), component list, interaction notes |
| `IMPLEMENTATION.md` | Engineers | Notes on implementation decisions, deviations, caveats. **Must end with a `## Handoff` section** when a downstream agent (Infra, QA, Security) will read it — see Handoff convention below. |
| `QA-REPORT.md` | QA | Test results, coverage gaps, validation status |
| `SECURITY-REPORT.md` | Security | Vulnerability findings, severity ratings, fix proposals, re-test results |
| `BLACK-HAT-REPORT.md` | Black Hat (via Security) | Outside-in attack hypotheses, verified findings, delta vs White Hat — Security integrates this |
| `SCALE-REPORT.md` | Scaler | Scale analysis at 100/1K/10K/1M thresholds, cost projections, inflection points |
| `OPTIMIZER-REPORT.md` | Optimizer | Solutions and efficiency improvements propagated; what was added, updated, or deferred |
| `SUMMARY.md` | Orchestrator | Status overview — what was built, how to verify, agent verdicts |
| `ACTION-LIST.md` | Orchestrator | Prioritized fix list across ALL reports — ordered by urgency, with file:line and exact fix per item. Only written when findings exist. This is the primary user-facing output. |
| `EVALS-REPORT.md` | Optimizer | Eval tasks harvested from this session (optional — only written when Eval Candidates found) |
| `AUDIT-REPORT.md` | Auditor | Inconsistencies found, canonical patterns extracted, agent files updated |

## Handoff Convention (Required for IMPLEMENTATION.md)

When a downstream agent (Infra, QA, Security) will read your `IMPLEMENTATION.md`, end the file with a `## Handoff` section. Downstream agents read this first — it lets them skip the full document for the two things they always need:

```markdown
## Handoff
- **Entry point**: `infra/cdk/lambda/my-feature/index.ts` (handler export: `handler`)
- **New env vars**: `MY_TABLE_NAME`, `FEATURE_FLAG`
- **CDK changes needed**: new NodejsFunction + API Gateway route + IAM policy for `my-table`
- **New DynamoDB access**: `my-table` — GetItem, PutItem (Infra must add to Lambda env + IAM)
- **Breaking changes**: none / yes — describe if yes
- **Test command**: `npm run test:cdk -- --testPathPattern=my-feature`
```

Omit fields that don't apply. If no downstream agent reads your output, this section is optional.

## File Index

### Languages
- [TypeScript](languages/typescript.md) — strict mode, AWS SDK v3, error patterns, naming
- [Python](languages/python.md) — type hints, device daemon patterns, async
- [Swift](languages/swift.md) — native iOS module patterns (brief)
- [Bash](languages/bash.md) — script safety rules

### Infrastructure
- [AWS](infra/aws.md) — Lambda, DynamoDB, S3, API Gateway, SES, Secrets Manager patterns
- [CDK](infra/cdk.md) — stack organization, construct patterns, env config
- [Docker](infra/docker.md) — image layering, multi-stage, non-root
- [CI/CD](infra/ci-cd.md) — GitHub Actions, build gates, deploy rules

### Standards
- [Testing](standards/testing.md) — unit/integration/e2e per layer, coverage rules
- [Security](standards/security.md) — OWASP baseline, secrets, IAM, auth patterns
- [API Design](standards/api-design.md) — REST conventions, error shapes, versioning
- [Git](standards/git.md) — commit format, branch naming, PR rules

### Solutions & Efficiency & Evals (read at startup)
- [Solutions Ledger](solutions.md) — growing index of error→solution pairs; skip any listed failed approach before trying it
- [Efficiency Ledger](efficiency.md) — growing index of token waste patterns and fixes; apply these to avoid known wasteful behaviors
- [Evals Ledger](evals.md) — growing bank of eval task definitions; Researcher reads to surface known hard cases; Optimizer writes when QA finds Eval Candidates
- [Patterns Ledger](patterns.md) — canonical code patterns discovered by the Auditor; all engineering and QA agents read at startup to build consistently
