# Test vs. live

Two isolated environments. Same dashboard, same SDK, different keys.

## Why two environments

* **Test** is for development. Verify with mock passports, iterate on rules and webhooks, never bill credits.
* **Live** is for production. Real documents, real proofs, real billing.

The environments don't share data, flows, sessions, or webhook endpoints.

## How they're separated

* **API keys** carry the environment in their prefix: `sk_test_...` vs. `sk_live_...`. A test key can only create sessions against a test flow; a live key only against a live flow.
* **Flows** are scoped to one environment.
* **Webhook endpoints** are per-environment. Set up `https://staging.example.com/webhooks/self` for test and `https://prod.example.com/webhooks/self` for live, with separate secrets.
* **Billing** never applies in test. Credit balance is for live only.

The environment is decided by the key you use: a `sk_test_…` key runs against test, a `sk_live_…` key against live.

## Mock passports

In test, you verify with a **mock passport** created in the Self app, with attributes you control (nationality, age, OFAC status), so you can exercise every branch of your handler without real users or real documents.

> Mock credentials are cryptographically distinct from real ones and never verify against a live flow.

See [Using mock passports](../guides/using-mock-passports.md) for the steps.

## Promoting a flow to live

Flows don't promote automatically. To take a tested configuration to production:

1. In the dashboard's **live** environment, create a new flow.
2. Copy the rule set, document list, and settings from the test flow.
3. Publish.
4. Update your backend to point at the live `flowId` (and use the live API key).

This is intentional, promoting a configuration is a deliberate act, not a side-effect of clicking around.

## Common test-mode patterns

### Run an integration test in CI

```ts
// In CI environment:
process.env.SELF_API_KEY = 'sk_test_...';
process.env.SELF_FLOW_ID = '<test flow uuid>';
process.env.SELF_WEBHOOK_SECRET = 'whsec_test_...';

// The test creates a session, simulates a mock-passport completion,
// and asserts the webhook handler fires with the expected payload.
```

The mock issuer can be driven via a test helper API (contact support if your team needs this).

### Run a staging environment

Use test keys + a test webhook endpoint, pointing at your staging backend. Real verification flows for your QA team, no production billing exposure.

## What to never do

* **Don't put a live key in a test environment.** Test environments tend to log requests, screenshot UIs, and store payloads in dev databases. A live key in any of those places is a security incident.
* **Don't reuse a webhook signing secret across environments.** Each endpoint gets its own. Mixing them is how you end up running test events against your prod database.
