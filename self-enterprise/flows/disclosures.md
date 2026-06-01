# Disclosures

A disclosure is what the user's ZK proof attests to. Every disclosure is a **predicate**: a true or false answer computed from the document without revealing the underlying value. You configure them per flow under [Configure → Rules](../dashboard/configure-a-product.md#rules), and which ones are available depends on the [product](../dashboard/configure-a-product.md#pick-a-product).

{% hint style="info" %}
The products return a **pass or fail outcome plus the predicate booleans you configured**. They do not return raw personal fields (no exact date of birth, no document number, no name). Field-level reveals are not part of these products today.
{% endhint %}

## Age

Available in **Pre KYC** and **Age Verification**.

You set a `minimumAge` (and optionally a `maximumAge`). The user proves they fall in range without revealing their birth date.

```json
{ "minimumAge": 18, "maximumAge": 65 }
```

The proof attests to a boolean outcome:

```json
{ "age_gte_18": true }
```

Use `minimumAge` alone for a simple floor (for example 18+). Add `maximumAge` for a band (for example 18 to 65).

## Country

Available in **Pre KYC**.

You allow or deny issuing countries. Use ISO 3166-1 alpha-3 codes, and pick one of include or exclude per flow, not both.

```json
{ "excludedCountries": ["PRK", "USA"] }
```

The user proves their document's issuing country satisfies the rule without revealing which country it is:

```json
{ "country_allowed": true }
```

## OFAC sanctions

Available in **Pre KYC**.

```json
{ "ofac": true }
```

The user proves their identity does not match the OFAC sanctions list. Self keeps the list updated daily. If the user matches, the verification comes back rejected and `ofac_clear` is omitted.

```json
{ "ofac_clear": true }
```

## Proof of human

Intrinsic to **Proof of Human**, and present implicitly in the other two, since every verification is backed by a genuine document.

The user proves they are a unique, real human. There is nothing to configure. Uniqueness uses a nullifier derived from the document, so each real-world identity maps to one stable, unlinkable identifier in your product. See [How verification works → Uniqueness without identity](../get-started/how-it-works.md#uniqueness-without-identity-nullifiers).

## What is never disclosed

* The user's name.
* Their exact date of birth (only the age predicate outcome).
* Their passport or document number.
* Their biometric data.
* Their address, and unless your flow uses country rules, anything about their country beyond the rule's pass or fail.

The proof attests to exactly what the flow's rules ask. Nothing else leaks.

## Related

* [Anatomy of a flow](anatomy.md).
* [Supported documents](supported-documents.md): which documents can satisfy which rules.
* [How verification works](../get-started/how-it-works.md): the model behind these predicates.
