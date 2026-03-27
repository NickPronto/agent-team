# Patterns Ledger

Canonical code patterns discovered by the team-auditor from real codebases. Engineers and QA
read this at startup so they build consistently from the first line of code.

- **Global patterns** live here — reusable across projects using the same stack
- **Project-specific patterns** (naming conventions, table prefixes, file structure) live in
  `.agent-team/PATTERNS.md` in the project root

**team-auditor** writes here. All engineering and QA agents read here at startup.

---

## Format

### <Pattern Name>
- **Convention**: one-sentence description
- **Canonical example**: `path/to/file.ts:line` — brief description
- **Code**: example from an actual codebase
- **Applies to**: agent names
- **Added**: YYYY-MM-DD

---

## AWS Lambda

### Module-Level SDK Client Initialization
- **Convention**: All AWS SDK clients are initialized at module level (outside the handler), never inside the handler function
- **Canonical example**: any Lambda handler with a `DynamoDBDocumentClient` at top of file
- **Code**:
  ```ts
  // CORRECT — module level, reused across warm invocations
  const dynamo = DynamoDBDocumentClient.from(new DynamoDBClient({ region: process.env.AWS_REGION }));

  export const handler = async (event) => { /* use dynamo */ };

  // WRONG — initialized on every invocation
  export const handler = async (event) => {
    const dynamo = DynamoDBDocumentClient.from(new DynamoDBClient({})); // ❌
  };
  ```
- **Applies to**: team-backend, team-infra
- **Added**: 2026-03-26

### Lambda Error Response Shape
- **Convention**: Error responses use `{ error: string }` — not `{ message: string }`
- **Canonical example**: standard error catch block in any Lambda handler
- **Code**:
  ```ts
  // CORRECT
  return { statusCode: 400, body: JSON.stringify({ error: 'Invalid input' }) };

  // WRONG
  return { statusCode: 400, body: JSON.stringify({ message: 'Invalid input' }) }; // ❌
  ```
- **Applies to**: team-backend, team-qa
- **Added**: 2026-03-26

### Secrets Manager — Lazy-Cached Credentials
- **Convention**: External credentials fetched from Secrets Manager are cached at module level with a nullable sentinel (`let cachedCreds: T | null = null`) and a getter function. Never fetched on every invocation.
- **Canonical example**: any Lambda that calls external APIs requiring credentials
- **Code**:
  ```ts
  let cachedCreds: MyCredentials | null = null;

  async function getCreds(): Promise<MyCredentials> {
    if (cachedCreds) return cachedCreds;
    const raw = await secretsClient.send(new GetSecretValueCommand({ SecretId: SECRET_NAME }));
    cachedCreds = JSON.parse(raw.SecretString!);
    return cachedCreds;
  }
  ```
- **Applies to**: team-backend
- **Added**: 2026-03-26

---

## DynamoDB

### Reserved Word Aliasing
- **Convention**: Fields named `status`, `name`, `type`, `date`, `value`, `data`, `key` MUST use ExpressionAttributeNames aliases in all expressions
- **Canonical example**: any query filtering on `status`
- **Code**:
  ```ts
  // CORRECT
  const result = await dynamo.send(new QueryCommand({
    TableName: TABLE,
    FilterExpression: '#status = :s',
    ExpressionAttributeNames: { '#status': 'status' },
    ExpressionAttributeValues: { ':s': 'active' },
  }));

  // WRONG — will throw ValidationException at runtime
  FilterExpression: 'status = :s'  // ❌
  ```
- **Applies to**: team-backend, team-qa
- **Added**: 2026-03-26

---

## API / Auth

### JWT Extraction Pattern
- **Convention**: Auth token is extracted from `Authorization: Bearer <token>` header. Missing or malformed header returns 401 immediately.
- **Canonical example**: any protected Lambda handler
- **Code**:
  ```ts
  const authHeader = event.headers?.Authorization || event.headers?.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return { statusCode: 401, body: JSON.stringify({ error: 'Unauthorized' }) };
  }
  const token = authHeader.slice(7);
  ```
- **Applies to**: team-backend, team-qa
- **Added**: 2026-03-26

---

## CDK / Infrastructure

### NodejsFunction — Canonical Lambda Construction
- **Convention**: All Lambda functions use `NodejsFunction` (not `lambda.Function` + `Code.fromAsset`). Entry points to `index.ts`, bundling excludes AWS SDK v3.
- **Canonical example**: any CDK stack defining a Lambda function
- **Code**:
  ```ts
  new NodejsFunction(this, 'FunctionId', {
    entry: path.join(__dirname, '../lambda/<name>/index.ts'),
    handler: 'handler',
    runtime: lambda.Runtime.NODEJS_20_X,
    timeout: cdk.Duration.seconds(30),
    memorySize: 256,
    bundling: {
      minify: true,
      sourceMap: false,
      target: 'es2022',
      externalModules: ['@aws-sdk/*'], // included in Lambda runtime
      forceDockerBundling: false,
    },
  });
  ```
- **Applies to**: team-infra
- **Added**: 2026-03-26

---

## Mobile

### Axios apiClient — Authenticated API Calls
- **Convention**: Mobile app uses Axios `apiClient` from `services/api.ts`, not raw `fetch()`. The client has auth interceptors that inject Bearer token and handle 401 refresh automatically.
- **Code**:
  ```ts
  import { apiClient } from '../services/api';
  const response = await apiClient.get('/clips');
  return response.data;
  ```
- **Applies to**: team-mobile
- **Added**: 2026-03-26

---

## Testing

### Test File Co-location
- **Convention**: Test files live alongside the code they test or in a `__tests__/` subdirectory at the same level — not in a top-level `test/` directory
- **Applies to**: team-qa, team-backend, team-frontend, team-mobile
- **Added**: 2026-03-26

### Test Structure — describe/it Nesting
- **Convention**: Top-level `describe` is the handler/component name. Nested `describe` for each logical group (happy path, error cases, auth). `it` statements describe the expected behavior, not the implementation.
- **Code**:
  ```ts
  describe('POST /leads handler', () => {
    describe('happy path', () => {
      it('returns 201 with lead id when input is valid', async () => { ... });
    });
    describe('error cases', () => {
      it('returns 400 when email is missing', async () => { ... });
    });
    describe('auth', () => {
      it('returns 401 when Authorization header is absent', async () => { ... });
    });
  });
  ```
- **Applies to**: team-qa, team-backend
- **Added**: 2026-03-26
