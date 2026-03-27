# CDK Standards (TypeScript)

## Stack Organization

One stack per domain concern. Organize stacks by domain:

```
infra/
  bin/
    app.ts          # CDK app entrypoint — instantiates all stacks
  lib/
    auth-stack.ts
    core-stack.ts
    lead-gen-stack.ts
    <domain>-stack.ts
```

## Stack Rules
- Stacks must be independently deployable — no circular cross-stack deps
- Cross-stack references: use `stack.exportValue()` / `Fn.importValue()` sparingly; prefer SSM Parameter Store for loose coupling
- Stack name: follow a consistent naming convention (e.g. `MyApp<Domain>Stack`)

## Constructs
- Use L2 constructs (e.g. `aws_lambda.Function`) over L1 CFn resources
- Group related resources into a single construct class when they're always deployed together
- Avoid `CfnOutput` for secrets — use Secrets Manager references

## Environment Config
- Never hardcode account IDs, region strings, or ARNs in construct code
- Pass via context: `app.node.tryGetContext('key')` or environment variables at synth time
- Prod vs dev: use environment-specific stack names, not feature flags inside stacks

## Lambda Constructs

```ts
const fn = new lambda.Function(this, 'MyFunction', {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset(path.join(__dirname, '../lambda/my-function')),
  environment: {
    TABLE_NAME: table.tableName,
    SECRET_NAME: secret.secretName,
  },
  timeout: Duration.seconds(30),
  memorySize: 256,
});

table.grantReadWriteData(fn);
secret.grantRead(fn);
```

Always use `.grant*()` methods — never write inline IAM policies by hand unless the grant method doesn't exist.

## DynamoDB Constructs

```ts
const table = new dynamodb.Table(this, 'Sessions', {
  tableName: 'myapp-sessions',
  partitionKey: { name: 'session_id', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: RemovalPolicy.RETAIN, // never DESTROY in prod
});
```

## API Gateway

```ts
const api = new apigateway.RestApi(this, 'Api', {
  restApiName: 'MyAppApi',
  defaultCorsPreflightOptions: {
    allowOrigins: apigateway.Cors.ALL_ORIGINS,
    allowMethods: apigateway.Cors.ALL_METHODS,
  },
});
```

## Removal Policies
- DynamoDB tables: `RemovalPolicy.RETAIN` always in production stacks
- S3 buckets: `RemovalPolicy.RETAIN` for data buckets
- Lambda functions: default (delete on stack destroy) is fine

## Testing CDK
- `cdk synth` must produce no errors before any deploy
- `npm run test:cdk` runs CDK unit tests — must pass before push
- CDK tests: use `Template.fromStack()` assertions, not snapshot tests
