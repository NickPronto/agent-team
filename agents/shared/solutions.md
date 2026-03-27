# Team Solutions Ledger

Known error→solution pairs discovered by agents during task execution.
Maintained by the Optimizer. Read at startup by every agent to avoid repeating failed approaches.

Format for each entry:
- **Error**: exact error message or symptom (quoted where possible)
- **Failed approach**: what was tried first
- **Solution**: what actually worked
- **Scope**: which agents/tools this applies to
- **Added**: date

---

## WebFetch / HTTP

### WebFetch 403 / 504 / Timeout on first attempt
- **Error**: `403 Forbidden`, `504 Gateway Timeout`, or fetch hangs >30s
- **Failed approach**: retrying the same URL
- **Solution**: immediately try an alternative source (different domain, WebSearch for mirror, cached version). Never retry the same URL. Use 60000ms timeout always.
- **Scope**: all agents that use WebFetch
- **Added**: 2026-03-26

### PDF URLs returning 403
- **Error**: `403` on direct PDF URL fetch
- **Failed approach**: fetching the PDF URL directly
- **Solution**: search for the HTML landing page or abstract page instead; extract key facts from that
- **Scope**: team-researcher, team-security
- **Added**: 2026-03-26

---

## AWS / DynamoDB

### DynamoDB ValidationException on reserved words
- **Error**: `ValidationException: Value provided in ExpressionAttributeNames must begin with #`  or `ValidationException: reserved word`
- **Failed approach**: using `status`, `name`, `type`, `date`, `value`, `data`, `key` directly in filter/key expressions
- **Solution**: always alias reserved words with ExpressionAttributeNames (e.g. `#status`, `#name`). Common reserved words: `status`, `name`, `type`, `date`, `value`, `data`, `key`
- **Scope**: team-backend, team-infra
- **Added**: 2026-03-26

### SecretsManager credentials stale after first invocation
- **Error**: `InvalidSignatureException` or auth errors on warm Lambda invocations after credential rotation
- **Failed approach**: fetching credentials inside the handler on every call
- **Solution**: cache credentials at module level with a TTL (e.g. 1 hour for OAuth tokens). Use `let cachedCreds: T | null = null` + getter function that checks TTL.
- **Scope**: team-backend
- **Added**: 2026-03-26

---

## npm / Node.js

### Prettier pre-commit hook rejection
- **Error**: `error  Insert X` or `Replace X` from Prettier pre-commit hook blocking commit
- **Failed approach**: committing without running Prettier first
- **Solution**: run `npx prettier --write <files>` on all modified `.ts`, `.tsx`, `.js`, `.jsx`, `.json` files before staging. Run after every code change, not just before commit.
- **Scope**: team-backend, team-frontend, team-mobile, team-infra
- **Added**: 2026-03-26

### `npm audit` blocking install on known low-severity CVEs
- **Error**: audit failure blocking `npm install` in CI
- **Failed approach**: `npm install --ignore-scripts` or skipping audit
- **Solution**: use `npm audit --audit-level=high` to only block on high/critical. Document accepted low/moderate CVEs in a comment.
- **Scope**: team-backend, team-frontend, team-mobile
- **Added**: 2026-03-26

---

## Mobile / React Native

### Metro required error during build
- **Error**: build fails because Metro bundler is not running
- **Failed approach**: using `expo start` or `eas build`
- **Solution**: use `expo prebuild` then `./gradlew assembleRelease` (Android) or `xcodebuild` (iOS). Set `SENTRY_DISABLE_AUTO_UPLOAD=true`. Never use Metro or EAS.
- **Scope**: team-mobile
- **Added**: 2026-03-26

---

## GitHub Actions / CI

### Workflow not triggering after push
- **Error**: workflow doesn't appear in Actions tab after push
- **Failed approach**: pushing again or editing workflow file
- **Solution**: check that workflow file YAML is valid (`yamllint`), branch filter matches the pushed branch, and the workflow is not disabled in the Actions tab UI
- **Scope**: team-infra
- **Added**: 2026-03-26

---

## AWS CDK CLI

### `cdk` command not found in shell
- **Error**: `Exit code 127 / command not found: cdk`
- **Failed approach**: running `cdk deploy` directly
- **Solution**: use `npx cdk <command>` — the CDK CLI is a local dev dependency, not globally installed. Always prefix with `npx` when running CDK commands in this repo.
- **Scope**: team-infra, orchestrator (main session)
- **Added**: 2026-03-27

---

## Python / Device Daemon

### pytest not discovering tests in device daemon directory
- **Error**: `no tests ran` despite test files existing
- **Failed approach**: running `pytest` from repo root
- **Solution**: run from the daemon directory with the `pytest.ini` present there, or use `pytest device/daemon/` with explicit path
- **Scope**: team-backend (daemon work)
- **Added**: 2026-03-26
