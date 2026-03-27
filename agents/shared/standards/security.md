# Security Standards

## Secrets Management
- **All secrets in AWS Secrets Manager** — never in env vars, config files, or source code
- Secret naming: `myapp/<service>/<name>` (e.g. `myapp/jwt-secret`)
- On device: API key stored at a root-only path (chmod 600) — never in world-readable config
- Lambda: read from Secrets Manager at cold start, cache at module level
- Never log secret values — log only `secretId` or `keyId` for debugging

## Input Validation
- Validate at system boundaries: API Gateway request bodies, Lambda event payloads
- Never trust client-supplied IDs to scope queries — always verify ownership
- DynamoDB: use parameterized expressions (ExpressionAttributeValues) always — never string interpolation

## Authentication
- User JWT: HS256, secret from Secrets Manager, verify on every protected request
- Admin JWT: HS256, separate secret, separate middleware
- Device API keys: one per device (compromise of one doesn't expose others), stored in API Gateway usage plan
- Never accept unsigned or self-signed JWTs in production

## IAM Least Privilege
- Lambda roles: one per function, grant only what it needs
- No `"Resource": "*"` on DynamoDB PutItem, DeleteItem, or S3 DeleteObject
- No `"Action": "*"` ever
- Service-to-service: use IAM roles, not access keys

## Response Security
- **Never return stack traces** or internal error details to API clients
- Error shape: `{ error: "human message", code: "ERROR_CODE" }` — no internal details
- **Never log external API response bodies** — they may contain PII or secrets
- Log only: status codes, error codes, sanitized identifiers

## OWASP Top 10 Checklist (per change)
- [ ] **Injection**: DynamoDB expressions parameterized? No string concatenation in queries?
- [ ] **Broken Auth**: JWT verified? Expiry checked? Signature validated?
- [ ] **Sensitive Data**: No secrets in logs, responses, or config files?
- [ ] **Broken Access Control**: Ownership verified before data access?
- [ ] **Security Misconfiguration**: CORS restricted? No wildcard IAM?
- [ ] **Vulnerable Dependencies**: No known CVEs in new packages?

## Dependency Policy
- Pin dependencies to exact versions in production Lambda packages
- Run `npm audit` before adding new packages
- Prefer AWS SDK v3 (modular, tree-shakeable) over v2
- No packages with known critical CVEs

## Data Classification
- User identifiers — treat with care; not PII by itself but don't expose unnecessarily
- Bank account info — use a third-party payments provider; never store raw account numbers
- JWT tokens — treat as passwords, never log
- Device location — treat as business-sensitive
