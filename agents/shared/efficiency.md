# Team Efficiency Ledger

Known token waste patterns and fixes discovered by agents during task execution.
Maintained by the Optimizer. Read at startup by every agent to avoid wasteful patterns.

Format per entry:
- **Pattern**: what wasteful behavior was observed
- **Fix applied**: what was changed in the agent file(s), or global rule added
- **Agent(s)**: which agent files were updated
- **Expected gain**: approximate token savings, or qualitative description
- **Added**: date

---

## Startup / Context Loading

### Load files conditionally based on task type
- **Pattern**: Agents load all reference docs at startup regardless of whether the task touches that domain (e.g. team-backend loading CDK docs for a pure Lambda task, team-researcher loading hardware docs for a software-only investigation)
- **Fix applied**: Global rule — before loading any reference doc beyond TEAM-CONFIG.md and CLAUDE.md, check whether the task actually touches that domain. If not, skip it.
- **Agent(s)**: all agents
- **Expected gain**: 500–2000 tokens per task depending on which docs are skipped
- **Added**: 2026-03-26

### Don't re-read files already in context
- **Pattern**: Agents read a file, then later in the same session read it again (e.g. reading TASK.md at startup and then reading it again before writing output)
- **Fix applied**: Global rule — if a file was read earlier in the session, reference it from memory. Only re-read if you need a specific section you didn't capture the first time.
- **Agent(s)**: all agents
- **Expected gain**: 200–800 tokens per re-read avoided
- **Added**: 2026-03-26

---

## Tool Calls

### Merge sequential greps into one
- **Pattern**: Running `grep patternA file` then immediately `grep patternB file` when both results are needed
- **Fix applied**: Global rule — when searching the same file or directory for multiple patterns, use a single grep with alternation (`patternA|patternB`) where possible
- **Agent(s)**: all agents
- **Expected gain**: 1 tool call per merged grep; adds up on large codebases
- **Added**: 2026-03-26

### Use Glob before Grep on unknown codebases
- **Pattern**: Running broad grep searches across the entire repo when a targeted Glob would narrow the search space first
- **Fix applied**: Global rule — when looking for a specific file type or name pattern, Glob first to find candidates, then Grep only within those files
- **Agent(s)**: team-researcher, team-backend, team-security
- **Expected gain**: Reduces grep result noise; avoids reading irrelevant matches
- **Added**: 2026-03-26

---

## Output

### Write workspace files once, not iteratively
- **Pattern**: Agent writes IMPLEMENTATION.md, then appends to it, then appends again as work progresses — resulting in multiple Write/Edit calls to the same file
- **Fix applied**: Global rule — draft the full output in memory, then write the workspace file once at the end. Use Edit only for targeted corrections, not iterative construction.
- **Agent(s)**: all implementation agents
- **Expected gain**: Reduces Write/Edit tool calls; cleaner workspace files
- **Added**: 2026-03-26
