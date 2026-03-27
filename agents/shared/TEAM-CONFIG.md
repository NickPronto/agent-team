# Team Shared Config

Global standards for the development team. Every agent reads this file on startup, then reads the project's `CLAUDE.md` for overrides. **Project rules always win.**

## Protocol

1. **Read your inbox**: check `.agent-team/<slug>/inbox/team-<name>.md` — if it exists, read it and incorporate any peer messages as additional context before starting your primary work.
2. Read this file on startup
3. Read `~/.claude/agents/shared/solutions.md` — **skip any approach listed as a failed approach before trying it**
4. Read the project's `CLAUDE.md` (overrides these globals)
5. Load only the referenced files relevant to your task — don't load everything
6. Write all task output to the workspace directory provided by the orchestrator
7. Never re-ask for context already in these files

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

## Peer Mesh Protocol

Agents operate as a delegating mesh — each agent spawns its natural downstream directly rather than returning to the orchestrator for every handoff. This section defines the three coordination mechanisms all agents use.

### Three Mechanisms

**1. Delegating spawns** — Every agent has the `Agent` tool. When your work is complete, spawn your natural downstream agent directly. Results bubble back up the call chain. Use this when you need the downstream result before the parent can continue, or when it's the natural next step in the workflow.

**2. Inbox protocol** — Any agent can write a non-blocking message to any peer's inbox at `.agent-team/<slug>/inbox/team-<name>.md`. Every agent reads its inbox at startup before doing its primary work. Use this when you notice something outside your domain that a peer would want to know about before they start — without blocking your own work.

**3. SESSION.md** — Agents append their status to `SESSION.md` in the workspace. Used to track what's done and prevent duplicate work in parallel scenarios.

### Tier Structure (spawn direction)

Agents may only spawn agents in the same tier or downstream (lower tier number = upstream). Agents may write inbox messages to any tier.

```
Tier 0: Orchestrator       (session start + final synthesis only)
Tier 1: Researcher, Designer, Scaler (cost audit)
Tier 2: Architect, Prompt Coach
Tier 3: Engineers — Backend, Infra, Mobile, Frontend, Firmware, ML, Hardware
Tier 4: QA, Optimizer
Tier 5: Security, Scaler (scale validation), Auditor
```

### Spawn vs Inbox Decision Rule

| Situation | Mechanism |
|---|---|
| Agent needs result before continuing | Spawn (blocking) |
| Natural next step in workflow | Spawn |
| Agent notices something outside its domain | Inbox (non-blocking) |
| Loop would be created by spawning | Inbox only |
| Parallel agents completing same tier | Return result to parent; parent coordinates downstream |

### Parallel Coordination Rule

When an orchestrator or agent spawns multiple agents in parallel (e.g. Backend + Frontend), those parallel agents DO NOT each spawn QA. They return their results to the spawning agent, which coordinates the next tier after all parallel agents complete. The spawn prompt will tell you: "You are running in parallel. Return your result when done — your parent will coordinate downstream."

### SESSION.md Format

```markdown
# Session: <task-slug>
| Agent | Status | Spawned By | Timestamp |
|---|---|---|---|
| team-researcher | COMPLETE | orchestrator | 14:00 |
| team-architect | IN_PROGRESS | team-researcher | 14:18 |
```

**On start**: append a row with `IN_PROGRESS` and the current time from `` `date '+%H:%M'` ``.
**On complete**: append a completion row with `COMPLETE` and current time.

Example bash command to get current time: `date '+%H:%M'`

### Inbox Message Format

```markdown
## [team-<sender> → team-<recipient>] <timestamp>
**Re**: <brief subject>
**Note**: <observation — what you noticed and why it might matter to the recipient>
**Blocking you**: No — context for your review pass.
```

Write inbox messages **before you spawn downstream agents**, so peers receive context before they start.

### Stretch Zone Principle

During your work, you may notice concerns that fall outside your primary domain. Use judgment: if a peer would want to know about it before they start their work, write to their inbox.

Examples of good stretch:
- Backend notices a manual auth check that looks wrong → writes to Security inbox
- Architect spots a design that will be hard to test → writes to QA inbox
- QA finds an architectural gap during testing → writes to Architect inbox
- Infra notices an IAM policy that's overly permissive → writes to Security inbox
- Backend notices a DynamoDB access pattern that will hot-partition at scale → writes to Scaler inbox

**Do not implement work outside your domain. Notice, flag, and let the relevant specialist handle it.**

## Retro Protocol

After every task, the orchestrator spawns `team-optimizer` in retro mode. The optimizer:
1. Spawns each participating agent in retro mode (they self-reflect, don't implement)
2. Synthesizes cross-agent patterns
3. Applies concrete edits directly to agent `.md` files
4. Writes `RETRO-REPORT.md` to the workspace

**Agents in retro mode** write `retro-team-<name>.md` to the workspace. They reflect on: misses, process waste, inbox/stretch zone gaps, and what they'd do differently.

**The optimizer applies learnings immediately** — agent files are updated before the next task runs. This is the primary mechanism by which the team improves over time.

**Workspace files added by retro:**
| File | Written by | Contains |
|---|---|---|
| `retro-team-<name>.md` | Each agent (retro mode) | Self-reflection and suggested file edits |
| `RETRO-REPORT.md` | Optimizer (retro mode) | Synthesis, updates applied, cross-agent patterns |

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
| `retro-team-<name>.md` | Each agent (retro mode) | Self-reflection on misses, process waste, and suggested agent file edits |
| `RETRO-REPORT.md` | Optimizer (retro mode) | Synthesis of all agent retros, cross-agent patterns, updates applied to agent files |

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
