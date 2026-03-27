# Claude Code Agent Team

A 17-agent development team for Claude Code that auto-routes tasks to the right specialist. Describe what you want to build — the orchestrator figures out who needs to work on it, in what order, and synthesizes the results.

## What This Is

This is a complete multi-agent system for Claude Code. When you type `/team` followed by a task description, an orchestrator agent reads your request, selects the right combination of specialist agents, and coordinates them through discovery, design, implementation, testing, and security review. Each agent is a Claude Code sub-agent with a focused role, a curated knowledge base, and clear output contracts.

The team runs as a **peer mesh** — not a hub-and-spoke. Agents spawn their natural downstream agents directly (Researcher → Architect → Engineers → QA → Security), communicate laterally via an inbox protocol, and self-improve after every task through a built-in retro loop.

The team is built around a shared knowledge infrastructure: a solutions ledger that accumulates error→solution pairs across sessions, an efficiency ledger that reduces wasted steps over time, an eval bank for regression testing, and a patterns ledger that captures canonical code conventions from your actual codebase.

## Agent Roster

| Agent | Role |
|---|---|
| **team-backend** | AWS Lambda handlers, DynamoDB, API integrations, device daemon (Python) |
| **team-infra** | CDK stacks, IAM policies, GitHub Actions, CloudWatch alarms |
| **team-mobile** | React Native / Expo — iOS and Android app features, BLE, IAP, release ops |
| **team-frontend** | Admin web panel — React dashboards, data tables, operator tools |
| **team-researcher** | Pre-implementation investigation — codebase patterns, library docs, risk surfacing |
| **team-architect** | System design, API contracts, data models, implementation plans |
| **team-prompt-coach** | Brief sharpening — converts vague requirements into testable acceptance criteria |
| **team-optimizer** | Propagates solutions and efficiency improvements across agent files automatically |
| **team-auditor** | Consistency detective — finds pattern drift, extracts canonical conventions |
| **team-designer** | UI/UX specs — ASCII wireframes, user flows, component contracts |
| **team-qa** | Test writing, test execution, spec compliance, regression coverage |
| **team-security** | White-hat penetration testing, vulnerability assessment, fix validation |
| **team-scaler** | Scale stress-testing (100/1K/10K/1M users) and AWS cost auditing |
| **team-blackhat** | Red-team adversarial agent — outside-in attack hypotheses, delta vs white-hat |
| **team-ml** | YOLOv8 training, RKNN model compilation, SageMaker pipelines, NPU inference |
| **team-firmware** | ESP32 firmware — BLE scanning, IMU, UART protocol, PlatformIO |
| **team-hardware** | PCB schematic review, BOM, routing guides, datasheet cross-referencing |

## How to Install

See [INSTALL.md](INSTALL.md) for the full step-by-step guide.

Quick version:
1. Clone this repo
2. Copy `agents/` to `~/.claude/agents/` (merge — don't overwrite existing files)
3. Copy `skills/team.md` to `~/.claude/commands/team.md`

## How to Use

In any Claude Code session, type:

```
/team <your task description>
```

Examples:

```
/team Add a referral code system — users get a discount when they invite a friend who makes a purchase

/team The clip list endpoint is returning 500 for users with more than 50 clips

/team Security audit of our auth flow and IAP verification

/team We need to support a new venue type — review the data model and add the required API endpoints

/team Review the codebase for consistency issues — we've had 4 engineers building in parallel
```

## How the Team Works

### Peer Mesh Architecture

Agents don't route everything through the orchestrator. Each agent spawns its natural downstream agent directly when its work is done. Researcher spawns Architect. Architect spawns Backend + Frontend in parallel. Backend spawns QA when its implementation is complete. The orchestrator only kicks off the first tier and returns at the end to synthesize.

This means less latency between steps and more natural handoffs — the agent that produced an artifact decides when it's ready and who needs it next.

### Inbox Protocol (Lateral Communication)

Every agent has an inbox file in the workspace. Any agent can write a non-blocking message to any peer's inbox at any time. Agents read their inbox on startup before beginning their primary work.

This enables cross-domain communication without blocking the call chain. A backend engineer notices a manual auth check that looks wrong — it writes to Security's inbox before spawning QA. Security arrives informed instead of starting cold.

### Stretch Zones

Agents are specialists, but they notice things outside their lane. Every agent is expected to apply judgment: *if a peer would want to know about this before they start, write to their inbox.*

Backend flags scale hotspots to Scaler. Architect flags testing complexity to QA. Infra flags over-permissive IAM to Security. QA flags architectural gaps it found during testing back to Architect. The result is higher initial quality — issues surface earlier in the chain rather than waiting for the specialist's dedicated pass.

### Retro Learning Loop

After every task, the orchestrator triggers the optimizer in retro mode. The optimizer spawns each participating agent with its own output file, the final SUMMARY, and the ACTION-LIST. Each agent reflects: what did I miss? what slowed me down? what would I do differently?

The optimizer synthesizes the reflections and applies **concrete, surgical edits directly to each agent's `.md` file** — no manual intervention. A `RETRO-REPORT.md` lands in the workspace recording every change applied.

The team gets measurably better after each task. Patterns that were missed become checklist items. Context that arrived late gets moved earlier. Dead-end approaches get added to the solutions ledger.

### What the Orchestrator Does

When you invoke `/team`, the orchestrator:

1. **Reads shared config** — loads `TEAM-CONFIG.md` and your project's `CLAUDE.md` (if present)
2. **Runs discovery** — asks up to 3 clarifying questions per round until the task is well-understood. Presents a plan/grill-me choice before proceeding.
3. **Selects models** — assigns Haiku/Sonnet/Opus to each agent based on task complexity (some agents always run Opus: architect, auditor, security, scaler, hardware)
4. **Creates a workspace** — `.agent-team/<date>-<slug>/` with `TASK.md`, `SESSION.md`, and an `inbox/` directory
5. **Spawns Tier 1 agents** — agents self-chain from there per the peer mesh protocol
6. **Monitors for blockers** — surfaces issues immediately, enforces a 2-cycle circuit breaker
7. **Triggers the optimizer** — after each agent, propagates error→solution pairs to the shared ledger
8. **Synthesizes** — writes `SUMMARY.md` and `ACTION-LIST.md` (prioritized fixes, if any)
9. **Runs retro** — in parallel with reporting results, triggers the retro learning loop

## Shared Knowledge Base

The `agents/shared/` directory contains the team's collective memory:

| File | Purpose |
|---|---|
| `TEAM-CONFIG.md` | Global standards every agent reads at startup |
| `solutions.md` | Error→solution pairs — agents skip failed approaches automatically |
| `efficiency.md` | Token waste patterns — agents avoid known wasteful behaviors |
| `evals.md` | Reusable eval task definitions from real QA sessions |
| `patterns.md` | Canonical code patterns extracted by the Auditor from real codebases |
| `languages/` | TypeScript, Python, Swift, Bash standards |
| `infra/` | AWS, CDK, Docker, CI/CD patterns |
| `standards/` | Testing, security, API design, Git conventions |

The solutions and efficiency ledgers grow automatically as agents work. Every error a agent encounters and solves gets written to `solutions.md` so future agents skip the dead end. Every wasteful step an agent notices gets written to `efficiency.md` so the team gets faster over time.

## Project-Specific Customization

- **`CLAUDE.md`** in your project root — project rules override all global agent behavior. Every agent reads this file. Use it for stack-specific constraints, build commands, service names, and anything that differs from the defaults.
- **`.agent-team/PATTERNS.md`** in your project root — project-specific canonical patterns written by the Auditor after it reads your codebase. Engineers read this at startup so they build consistently.
- **`~/.claude/agents/shared/TEAM-CONFIG.md`** — edit the `Primary Stack` table to match your tech stack.

## Workspace Outputs

Every task produces a workspace at `.agent-team/<date>-<slug>/` containing:

- `TASK.md` — what was planned and which agents ran
- `RESEARCH.md` — Researcher's findings (if research ran)
- `BRIEF.md` — Prompt Coach's sharpened brief (if coach ran)
- `ARCHITECTURE.md` — Architect's design (if architect ran)
- `IMPLEMENTATION.md` — what was built and how
- `QA-REPORT.md` — test results, coverage, spec compliance
- `SECURITY-REPORT.md` — vulnerability findings and severity ratings
- `SCALE-REPORT.md` — scale analysis and cost projections
- `SESSION.md` — real-time agent status board (IN_PROGRESS / COMPLETE)
- `inbox/` — peer-to-peer messages written during the task
- `SUMMARY.md` — final status overview
- `ACTION-LIST.md` — prioritized fix list (only if findings exist)
- `retro-team-<name>.md` — each agent's self-reflection
- `RETRO-REPORT.md` — optimizer's synthesis and list of agent file edits applied

## License

MIT
