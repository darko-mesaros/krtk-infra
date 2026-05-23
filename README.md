# krtk-infra

AWS CDK infrastructure for the [krtk.rs](https://krtk.rs) URL shortener.

## Structure

```
├── bin/krtk-rs.ts           # CDK app entry point
├── lib/
│   ├── krtk-rs-stack.ts     # Main stack (Lambda, API GW, DynamoDB, CloudFront)
│   ├── certificate-stack.ts # ACM certificate (us-east-1 for CloudFront)
│   └── secrets-stack.ts     # Google API key secret
├── website/                 # Static frontend (deployed to S3)
└── test/                    # CDK tests
```

## Related Repo

Lambda function source code lives in the sibling repo: **krtk-lambdas**

Both repos should be cloned as siblings:

```
repos/
├── krtk-lambdas/   # Rust Lambda functions
└── krtk-infra/     # this repo
```

The CDK stack uses `cargo-lambda-cdk` and references the Rust crates via:
```typescript
manifestPath: '../krtk-lambdas/lambda/create_link/Cargo.toml'
```

## Development

```bash
# Install dependencies
npm install

# Compile TypeScript
npx tsc

# List stacks
npx cdk ls

# Deploy main stack
just deploy

# Invalidate CloudFront cache
just invalidate-cache
```

## Stacks

| Stack | Region | Purpose |
|-------|--------|---------|
| CertificateStack | us-east-1 | ACM cert for CloudFront |
| SecretsStack | us-west-2 | Google Safe Browsing API key |
| KrtkRsStack | us-west-2 | Everything else (Lambda, API GW, DynamoDB, CloudFront, S3) |

## Architecture

CloudFront → API Gateway (HTTP API) → Lambda (Rust) → DynamoDB

CloudFront real-time logs → Kinesis → process_analytics Lambda → DynamoDB (visit counts)
