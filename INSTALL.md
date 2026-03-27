# Install Guide

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed and configured
- A Claude Code project or working directory

## Step 1 — Clone the repo

```bash
git clone https://github.com/your-org/agent-team-repo.git
cd agent-team-repo
```

## Step 2 — Copy the agents directory

The `~/.claude/agents/` directory may already exist if you have other agents installed. **Merge — do not overwrite.**

If `~/.claude/agents/` does not exist yet:
```bash
cp -r agents/ ~/.claude/agents/
```

If `~/.claude/agents/` already exists (merge carefully):
```bash
# Copy agent files
cp agents/*.md ~/.claude/agents/

# Copy shared directory — merge subdirectories
mkdir -p ~/.claude/agents/shared/languages
mkdir -p ~/.claude/agents/shared/infra
mkdir -p ~/.claude/agents/shared/standards

cp agents/shared/*.md ~/.claude/agents/shared/
cp agents/shared/languages/*.md ~/.claude/agents/shared/languages/
cp agents/shared/infra/*.md ~/.claude/agents/shared/infra/
cp agents/shared/standards/*.md ~/.claude/agents/shared/standards/
```

**Warning**: If you already have a `solutions.md`, `patterns.md`, or `efficiency.md` in `~/.claude/agents/shared/`, review the incoming files before overwriting — your existing ledgers may contain project-specific knowledge you want to keep. Consider manually merging them.

## Step 3 — Copy the team skill

The `/team` command lives in `~/.claude/commands/`. Create the directory if it doesn't exist:

```bash
mkdir -p ~/.claude/commands
cp skills/team.md ~/.claude/commands/team.md
```

If you already have a `~/.claude/commands/team.md`, back it up first:
```bash
cp ~/.claude/commands/team.md ~/.claude/commands/team.md.bak
cp skills/team.md ~/.claude/commands/team.md
```

## Step 4 — Verify the install

Open a Claude Code session and type:

```
/team
```

You should see the orchestrator activate and ask you what you'd like to build. If Claude doesn't recognize the command, check that `~/.claude/commands/team.md` exists and is readable.

## Step 5 — Use the team

In any Claude Code session, type `/team` followed by your task:

```
/team Add a password reset flow to the auth system

/team The checkout API is returning 500 — investigate and fix

/team Security audit of our JWT handling and IAP verification

/team Refactor the user profile module — it's grown too large
```

The orchestrator will ask discovery questions if the task needs clarification, then route to the right agents.

---

## Customizing for Your Project

### Tell agents about your stack

Create a `CLAUDE.md` in your project root. Every agent reads this file on startup — it overrides all global defaults. Include:

- Tech stack (language versions, frameworks, cloud provider)
- Build and test commands (`npm run test:web`, `./gradlew assembleRelease`, etc.)
- Key constraints ("never use Metro", "Prettier required before commit", etc.)
- Service and table names agents need to know
- Any domain-specific rules

Example:
```markdown
# CLAUDE.md

## Stack
- TypeScript (strict), Node.js 20, React Native 0.74
- Backend: AWS Lambda + DynamoDB + CDK
- Mobile: Expo (local builds only — never Metro or EAS)

## Commands
- Tests: `npm run test:web` and `npm run test:cdk`
- Format: `npx prettier --write <files>` (enforced pre-commit)
- Android build: `cd android && ./gradlew assembleRelease`

## Constraints
- DynamoDB table prefix: `myapp-`
- All secrets in AWS Secrets Manager: `myapp/<service>/<name>`
- Never log API response bodies (may contain PII)
```

### Update the primary stack in TEAM-CONFIG

Edit `~/.claude/agents/shared/TEAM-CONFIG.md` and update the `Primary Stack` table to reflect your actual tech stack. Agents use this table to understand what language, framework, and database they're working with.

### Add project-specific patterns

After running the Auditor (`/team run a consistency audit`) on your codebase, it will create `.agent-team/PATTERNS.md` in your project root with patterns extracted from your actual code. All engineering and QA agents read this file at startup. You can also add patterns manually — use the format documented in `~/.claude/agents/shared/patterns.md`.

### Customize individual agents

Each agent file in `~/.claude/agents/` is a plain Markdown file. You can edit any agent's:
- `<role>` block — to change how the agent understands its purpose
- `<startup>` block — to add or remove files it reads at startup
- `<key_patterns>` block — to add project-specific code patterns
- `<rules>` block — to add hard constraints

Changes take effect immediately on the next task.

### Disable agents you don't need

If your project doesn't use hardware, ML, or firmware, you can safely ignore those agents — the orchestrator won't route to them unless the task explicitly requires them. You don't need to delete the files.

---

## Troubleshooting

**`/team` not recognized**: Check that `~/.claude/commands/team.md` exists. Claude Code looks for skills in this directory.

**Agent not found error**: Check that the agent `.md` file exists in `~/.claude/agents/`. Agent names in the orchestrator routing table must match the `name:` field in the agent's frontmatter.

**Solutions ledger growing too large**: The `solutions.md` and `efficiency.md` files grow over time. They're plain Markdown — you can review and prune entries that are no longer relevant. Never delete the section headers, only individual entries.

**Discovery loop feels too long**: Append "just run it", "go ahead", or "no questions" to your `/team` prompt to skip discovery and proceed directly with the plan.
