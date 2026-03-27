# Testing Standards

## Gate: Must Pass Before Push
```bash
npm run test:web    # web app + Lambda unit tests
npm run test:cdk    # CDK infrastructure tests
```
Both must pass. Fix failures — never bypass pre-push hook.

## Testing Pyramid

| Layer | Type | Tool | Scope |
|---|---|---|---|
| Lambda handlers | Unit | Jest | Pure logic, no external calls |
| Lambda handlers | Integration | Jest + real DynamoDB | Full handler against real table |
| CDK stacks | Unit | Jest + CDK assertions | Template assertions |
| React Native | Unit | Jest + React Testing Library | Component rendering, hooks |
| React Native | E2E | Detox (future) | Not in CI currently |
| Device daemon | Unit | pytest | Pure functions |

## Lambda Testing Rules

**No database mocks in integration tests.** Mock/prod divergence has caused production failures (e.g. mocked tests passed, real migration failed). Integration tests must hit a real DynamoDB table (test table, not prod).

```ts
// Good — integration test hits real table
it('creates a session', async () => {
  const event = buildEvent({ ... });
  const result = await handler(event);
  expect(result.statusCode).toBe(201);
  // verify in real DynamoDB
  const item = await dynamo.send(new GetCommand({ TableName: TEST_TABLE, Key: { ... } }));
  expect(item.Item).toBeDefined();
});
```

**Every new Lambda handler needs at minimum:**
1. Happy path test
2. Error path test (bad input, missing fields)
3. Auth failure test (if endpoint is protected)

## CDK Test Pattern

```ts
import { Template } from 'aws-cdk-lib/assertions';

test('Lambda has correct memory', () => {
  const app = new cdk.App();
  const stack = new MyStack(app, 'TestStack');
  const template = Template.fromStack(stack);
  template.hasResourceProperties('AWS::Lambda::Function', {
    MemorySize: 256,
  });
});
```

Use `Template.hasResourceProperties()` assertions — not snapshot tests (snapshots break on every CFn change).

## React Native Testing

- Test components in isolation with mocked navigation and stores
- Test hooks separately from components
- Don't test implementation details — test behavior visible to the user
- Avoid `act()` warnings — they signal improper async handling

## Coverage Expectations
- New features: aim for 80%+ on new files
- Bug fixes: add a test that would have caught the bug
- Don't chase 100% — untestable glue code doesn't need tests
- Coverage report: `npm run test:web -- --coverage`
