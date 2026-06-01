# Configure a product

A flow always starts by choosing one of the three products. The product decides which rules you can set. You then configure the flow in the **Configure** tab of the product editor.

> 📸 _**Screenshot placeholder:** the "New configuration" screen showing the three products (Pre KYC, Age Verification, Proof of Human)._

## Pick a product

| Product | What the user proves | Configurable rules | Outcome |
| --- | --- | --- | --- |
| **Pre KYC** | Holds a genuine document, optionally meets an age floor, comes from an allowed country, clears OFAC | Minimum age, country allow or deny lists, OFAC | approved / rejected |
| **Age Verification** | Meets an age threshold | Minimum age (and optional maximum age) | approved / rejected |
| **Proof of Human** | Is a unique, real human | None (uniqueness is intrinsic) | verified / failed |

> Per-document disclosures (revealing a specific field like an exact date or document number) are not part of these products. Each one returns a pass or fail result plus the predicate outcomes you configured, never raw personal data.

## Rules

Rules are the predicates the user's proof must satisfy. The user proves each one without revealing the underlying value. Which rules are available depends on the product you picked:

| Rule | What it checks | Available in |
| --- | --- | --- |
| **Minimum age** | User is at least N years old. Never reveals the date of birth. | Pre KYC, Age Verification |
| **Maximum age** | User is at most N years old (combine with minimum for a band). | Age Verification |
| **Country rules** | Issuing country is on your allow list, or not on your deny list. ISO 3166-1 alpha-3 codes. | Pre KYC |
| **OFAC** | User does not match the OFAC sanctions list. Self keeps the list updated daily. | Pre KYC |

Proof of Human has no configurable rules: uniqueness comes from the document itself, so you just deploy it.

> 📸 _**Screenshot placeholder:** the Rules editor for a Pre KYC flow with a minimum age, a country deny list, and OFAC enabled._

The dashboard validates the combination before you can publish. For example, you cannot set both an allow list and a deny list for countries on the same flow.

## Documents

Which credential types you accept. The dashboard shows everything supported and you tick the ones you want.

* **Biometric passport**: chip-equipped ICAO 9303 e-passports. Highest assurance.
* **Aadhaar**: India's national ID. See the [document spec](../reference/document-specifications/aadhaar.md).
* **KYC attestation**: a partner-issued KYC credential. See the [KYC spec](../reference/document-specifications/kyc.md).

> **Coverage:** different documents cover different countries. See [Supported countries](../reference/supported-countries.md).

## Settings

Per-flow operational settings:

* **Success URL**: where the user goes after a successful verification.
* **Failure URL**: where the user goes if the verification fails (wrong document, a rule not met, expired).
* **Display name**: what the user sees on the hosted page (for example "Acme Marketplace").
* **Branding**: logo and accent color for the hosted page.

These are read at session creation time. You can override `successUrl` and `failureUrl` per session through the API when the destination varies (for example deep links).

## Draft vs. published

Editing the Configure tab updates a **draft**. The draft is not live until you open the [Deploy](publish-a-flow-version.md) tab and publish it.

In-flight sessions keep using the version they were created against, so publishing a new version never breaks an open session.

## Related

* [Anatomy of a flow](../flows/anatomy.md): the data model behind these tabs.
* [Disclosures](../flows/disclosures.md): what each rule proves.
* [Publish a flow version](publish-a-flow-version.md).
