# CI/CD Standards (GitHub Actions)

## General Rules
- Every PR must pass tests before merge
- Never skip hooks (`--no-verify`) — fix the underlying issue
- Secrets: always from GitHub Actions Secrets, never hardcoded in workflow files
- Pin third-party actions to a commit SHA, not a tag (`uses: actions/checkout@v4` is acceptable for first-party)

## Pre-Commit (local)
Prettier is enforced by the pre-commit hook. **Run before staging:**
```bash
npx prettier --write <modified-files>
```
Hook will reject unformatted `.ts`, `.tsx`, `.js`, `.jsx`, `.json` files.

## Pre-Push (local)
Both must pass before a push goes through:
```bash
npm run test:web
npm run test:cdk
```
Fix failures — do not bypass.

## Mobile Builds

**Never use Metro or EAS** (`expo start`, `eas build`, `eas submit`) — local builds only.

```bash
# Android
cd android && ./gradlew assembleRelease
# or
./scripts/build-android.sh

# iOS
xcodebuild -workspace ios/App.xcworkspace -scheme App -configuration Release
```

Required env vars for all builds:
```
SENTRY_DISABLE_AUTO_UPLOAD=true
```

Android device builds: set `reactNativeArchitectures=arm64-v8a` in `gradle.properties` to reduce build time and memory for single-architecture targets.

## Version Numbers
- `APP_VERSION` in `app.config.ts` — only bump when submitting a new App Store release
- Build numbers: automatic via `github.run_number` — never manually set

## Workflow Structure
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:web
      - run: npm run test:cdk

  deploy:
    needs: test  # always gate deploy on tests
    if: github.ref == 'refs/heads/main'
    ...
```

## Deployment Gates
- `main` branch only deploys to production
- Feature branches: test + build, no deploy
- Hotfixes: same pipeline, no exceptions
- CDK deploy: always `cdk diff` in CI, `cdk deploy` only on main after tests pass
