# Credits and usage

How we meter what you use and turn it into a bill.

## The credit model

Usage is denominated in **credits**, an abstract unit. Each plan includes some credits per month, and your USD price-per-credit is set on your plan's rate card.

> Credits are not USD cents. The exchange rate is part of your contract; check **Settings → Billing → Plan card** for your current rate.

This lets us:

* Adjust per-product pricing without touching your contract.
* Run promos and waivers in credits.
* Set spending caps independently of session volume.

## Per-product cost

Each product has a per-verification credit cost, baked in at session creation time (not at completion):

| Product | Credit cost (illustrative) |
| --- | --- |
| Pre KYC | 1 |
| Age Verification | 2 |
| Proof of Human | 3 |

(Concrete costs live in your dashboard. The numbers above are illustrative.)

The cost is reserved against your credit balance the moment a session is created. Sessions that end `expired` or `error` are **not billed**, you only pay for verifications the user actually completed.

## What counts as "consumed"

| Session terminal status | Consumed? |
| --- | --- |
| `valid` | Yes |
| `invalid` (predicate failed) | Yes |
| `error` (technical failure) | No, not billed |
| `expired` (user never finished) | No, not billed |

You pay for verification work, not for sessions the user never got to.

## Credit balance lifecycle

```
[plan grant @ cycle start] → [reserved on session create] → [not billed on expired/error]
                                         │
                                         ▼
                          [topped up via overage purchase, if enabled]
```

Watch the balance in **Settings → Billing → Credit balance**. It auto-refills at the start of each cycle.

## Insufficient credits

Each org has a **credit gate** with two modes (set under **Settings → Billing**):

* **`hard`** (default): if your balance is too low to cover a session's cost, `sessions.create(...)` is rejected with HTTP `402` (the SDK throws `SelfApiError` with `status: 402`). The session is not created. The error `details` include `balance`, `required`, and `planTier`.
* **`soft`**: sessions are still created when you're out of credits, so verification isn't interrupted, you reconcile the overage on your invoice. Use this when uninterrupted verification matters more than a hard spend cap.

To avoid this in production:

* Set up **low-balance alerts** under **Notifications**.
* Configure **overage** so we automatically purchase top-up credits when you cross zero (available on Starter and Enterprise plans).
* Watch the **Credits consumed** chart in Billing to forecast burn.

## Reading usage programmatically

Metered events are aggregated by our usage system. Customers on Enterprise plans get programmatic access to a usage API; reach out to your CSM.

For Starter and Free, the dashboard's CSV export is the source of truth.

## Test environment

Test verifications never consume credits and never appear on invoices. They're not even tracked against rate-limit windows. Use test freely.

## Examples

### Sizing for a launch

You expect 50,000 Age Verification checks in the first month (2 credits each):

* Expected consumption: 50,000 × 2 = 100,000 credits.
* If your plan includes 40,000, you'll need 60,000 in overage, so verify your overage rate before launch.

### Diagnosing a spike

The **Credits consumed** chart shows daily burn. If you see a spike, drill into the [Activity log](../dashboard/activity-log.md) for the same day, the External UUID column usually identifies the culprit (a stuck retry loop, a bot, a buggy integration).

## Related

* [Plans](plans.md).
* [Dashboard: Billing](../dashboard/billing.md).
