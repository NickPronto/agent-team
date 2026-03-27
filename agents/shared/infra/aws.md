# AWS Standards

## Region
Default: `us-west-2`. Always specify explicitly — never rely on ambient AWS config.

## Lambda

**Client initialization** — always at module level, never inside handler:
```ts
const dynamo = DynamoDBDocumentClient.from(new DynamoDBClient({ region: 'us-west-2' }));
const secrets = new SecretsManagerClient({ region: 'us-west-2' });
```

**Credentials** — lazy-loaded, cached at module level:
```ts
let creds: AppSecret | null = null;
async function getCreds(): Promise<AppSecret> {
  if (creds) return creds;
  const r = await secrets.send(new GetSecretValueCommand({ SecretId: process.env.SECRET_NAME! }));
  creds = JSON.parse(r.SecretString!);
  return creds;
}
```

**Environment variables** — always validate at startup:
```ts
const TABLE = process.env.TABLE_NAME ?? (() => { throw new Error('TABLE_NAME required'); })();
```

**Error responses** — consistent shape:
```ts
return { statusCode: 400, body: JSON.stringify({ error: 'message', code: 'VALIDATION_ERROR' }) };
```

## DynamoDB

**Reserved words** — must use ExpressionAttributeNames aliases:
Reserved: `status`, `name`, `type`, `date`, `value`, `data`, `key`, `size`, `state`, `time`, `count`

```ts
await dynamo.send(new UpdateCommand({
  TableName: TABLE,
  Key: { pk },
  UpdateExpression: 'SET #status = :s, #name = :n',
  ExpressionAttributeNames: { '#status': 'status', '#name': 'name' },
  ExpressionAttributeValues: { ':s': 'active', ':n': name },
}));
```

**Table naming**: use a consistent prefix (e.g. `myapp-<noun>`)
**GSI naming**: `<field>-index` (e.g. `user-index`, `date-index`)
**Keys**: use consistent `pk`/`sk` or domain-specific names

## Secrets Manager

- One secret per logical credential group: `myapp/<service>/<name>`
- Secret value: JSON string parsed on first use
- Never pass secrets as Lambda env vars — read from Secrets Manager
- Never log secret values

## S3

- Uploads: use presigned URLs (generated server-side, given to client)
- Downloads: serve via CloudFront signed URLs, not direct S3
- Bucket naming: `myapp-<purpose>-<account-id>`
- Object keys: `<category>/<id>/<filename>` (e.g. `clips/session123/clip.mp4`)

## API Gateway

**Auth patterns:**
- User endpoints: JWT Bearer (HS256, secret from Secrets Manager)
- Admin endpoints: JWT Bearer (HS256, separate secret, separate middleware)
- Device endpoints: API key header (`x-api-key`), one key per device
- Public endpoints: no auth (lead gen, health checks)

**CORS**: set explicitly on all endpoints returning JSON to browser clients

## SES (Email)

Requires both env vars to be set:
```ts
const SES_CONFIG_SET = process.env.SES_CONFIGURATION_SET!;
const FROM_EMAIL = process.env.FROM_EMAIL!;
```
Missing either causes silent failure or SDK error. Always validate at startup.

## IAM

- Least privilege — grant only the specific actions needed
- No `*` on resource ARN for destructive actions (DeleteItem, DeleteObject)
- Lambda execution roles: separate role per Lambda, not shared
- Never hardcode access keys — use IAM roles exclusively in Lambda

## External API Fail-Open Pattern

For Apple/Google IAP verification: fail open on 5xx (allow purchase through):
```ts
if (!res.ok && res.status >= 500) {
  console.warn(`External API unavailable: ${res.status}`);
  return allowPurchase(); // intentional fail-open
}
```
Do NOT add aggressive retries on external IAP APIs — this blocks purchases.
