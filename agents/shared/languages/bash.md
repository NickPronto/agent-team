# Bash Script Standards

## Safety
- Always start scripts with: `set -euo pipefail`
- `set -e` — exit on error
- `set -u` — exit on undefined variable
- `set -o pipefail` — pipe failures propagate

## Variables
- Quote all variable expansions: `"$VAR"`, `"${VAR}"`, `"$@"`
- Local variables in functions: `local var="value"`
- Constants: `readonly CONST="value"` at top of script
- Check required env vars early: `${REQUIRED_VAR:?Error: REQUIRED_VAR not set}`

## Input Validation
- Validate all required args at the top before doing any work
- Print usage and exit 1 on bad input: `echo "Usage: $0 <arg>" >&2; exit 1`

## Functions
```bash
my_function() {
  local arg1="$1"
  local arg2="$2"
  # body
}
```

## AWS CLI in Scripts
- Always specify `--region` explicitly — never rely on ambient config
- Capture output to variables, check return codes
- Use `--output json` + `jq` for parsing, not text scraping

## Common Patterns
```bash
# Check command exists
command -v jq >/dev/null 2>&1 || { echo "jq required" >&2; exit 1; }

# Temp files: clean up on exit
TMP=$(mktemp)
trap "rm -f $TMP" EXIT
```
