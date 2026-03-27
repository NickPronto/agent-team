# Git Standards

## Commit Message Format (Conventional Commits)
```
<type>(<scope>): <short summary>

[optional body]

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

**Types:**
- `feat:` — new feature
- `fix:` — bug fix
- `chore:` — maintenance, deps, config (no production code change)
- `refactor:` — code change that's neither a fix nor a feature
- `docs:` — documentation only
- `test:` — adding or fixing tests
- `ci:` — CI/CD pipeline changes
- `perf:` — performance improvement

**Rules:**
- Summary line: 72 chars max, imperative mood ("add X", not "added X")
- No period at end of summary
- Body: explain *why*, not what (the diff shows what)
- Scope (optional): module or area affected (`auth`, `mobile`, `cdk`, `daemon`)

## Branch Naming
```
feature/<short-description>
fix/<issue-or-description>
chore/<description>
hotfix/<description>
```

Examples: `feature/learning-system`, `fix/dynamo-reserved-words`, `hotfix/auth-token-expiry`

## Pre-Commit Checklist
- [ ] Run Prettier: `npx prettier --write <changed-files>`
- [ ] No secrets, credentials, or API keys in diff
- [ ] No debug `console.log` or `print()` statements left in
- [ ] Unrelated changes are not included (focused commits)

## Pre-Push Checklist
- [ ] `npm run test:web` passes
- [ ] `npm run test:cdk` passes
- [ ] Branch is up to date with main (rebase, don't merge)

## Pull Requests
- One concern per PR — don't bundle unrelated changes
- Title: same format as commit summary (`feat(auth): add admin JWT refresh`)
- Description: what changed, why, how to test
- Keep PRs small — large PRs get missed bugs
- Rebase onto main before requesting review (no merge commits in PR history)

## What Not to Commit
- `.env` files, `credentials/`, `*.p8`, `*.pem`, `*.key`
- `node_modules/`, build artifacts, `.DS_Store`
- Large binary files (videos, PDFs) — use `.gitignore`
- Config files with real environment values (use templates with `.example` suffix)
