# TypeScript Standards

## Compiler
- `strict: true` in tsconfig — no exceptions
- `noImplicitAny`, `strictNullChecks`, `noUncheckedIndexedAccess` enabled
- Target: `ES2022`, module: `NodeNext` for Lambda; `ESNext` for React Native

## Types
- **Never use `any`** — use `unknown` + type guard, or a proper interface
- Interfaces for object shapes; `type` for unions, intersections, and primitives
- Avoid type assertions (`as T`) unless you can prove safety — add a comment if you use one
- Prefer `readonly` on properties that shouldn't change after construction

## Naming
- `PascalCase` for types, interfaces, classes, React components
- `camelCase` for variables, functions, parameters
- `SCREAMING_SNAKE_CASE` for true constants (env var names, magic numbers)
- `kebab-case` for file names

## Exports
- Named exports everywhere except React components (which may use default export)
- No barrel files that re-export everything from a directory — they cause circular deps

## AWS SDK v3
- **Initialize all SDK clients at module level** (outside the handler function)
- This reuses connections across warm Lambda invocations — never instantiate inside a handler

```ts
// Good
const client = new DynamoDBDocumentClient(new DynamoDBClient({}));
export const handler = async (event) => { ... };

// Bad
export const handler = async (event) => {
  const client = new DynamoDBDocumentClient(...); // instantiated on every cold start
};
```

## Credentials & Secrets

Cache credentials at module level with lazy loading:

```ts
let cachedSecret: MySecret | null = null;

async function getSecret(): Promise<MySecret> {
  if (cachedSecret) return cachedSecret;
  const res = await secretsClient.send(new GetSecretValueCommand({ SecretId: SECRET_NAME }));
  cachedSecret = JSON.parse(res.SecretString!);
  return cachedSecret;
}
```

## DynamoDB Reserved Words

`status`, `name`, `type`, `date`, `value`, `data`, `key`, `size`, `state`, `time` are reserved.
Always alias them in expressions:

```ts
FilterExpression: '#status = :s',
ExpressionAttributeNames: { '#status': 'status' },
ExpressionAttributeValues: { ':s': 'active' },
```

## Error Handling

- Throw on non-ok HTTP responses — do not silently swallow
- **Never log response bodies from external APIs** (may contain secrets/PII)
- Log `res.status` and `res.statusText` only
- Return typed error responses at Lambda boundaries: `{ error: string, code?: string }`
- Use `instanceof` checks or discriminated unions — no string matching on `error.message`

## Async
- `async/await` only — no raw Promise chains, no callbacks
- Always `await` — never fire-and-forget unless explicitly intentional (add a comment)
- Handle rejections — unhandled promise rejections crash Lambda

## Formatting
- Prettier is enforced by pre-commit hook — run `npx prettier --write <files>` before staging
- Single quotes, 2-space indent, trailing commas (ES5), 100-char line width
