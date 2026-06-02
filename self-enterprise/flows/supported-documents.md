# Supported documents

A user verifies with whichever supported document they hold, there's no per-flow document picker. What differs is the geographic footprint and which rules each document can satisfy.

## Biometric passport (ICAO 9303 e-passport)

**Highest assurance.** A passport-chip read produces a credential signed by the issuing state.

* **Coverage:** 60+ countries. See [Supported countries](../reference/supported-countries.md).
* **Can satisfy:** every rule, age, excluded countries, OFAC, and proof of human.
* **UX:** the user holds the passport to the phone's NFC reader. About 30 seconds end to end.
* **Notes:** needs an NFC-capable phone (most iPhones and Android 8+).

## Aadhaar (India)

India's universal government ID.

* **Coverage:** India only.
* **Can satisfy:** age and proof of human. Does **not** support country rules (Aadhaar doesn't encode nationality the same way) or OFAC directly.
* **UX:** the user scans the QR code on their Aadhaar and provides their share code.
* **Notes:** see the [Aadhaar spec](../reference/document-specifications/aadhaar.md) for the cryptographic detail.

## KYC attestation (partner-issued)

A credential issued by a Self partner KYC provider after a remote KYC check.

* **Coverage:** depends on the issuer. Most partners cover the US, EU, UK, and a long tail.
* **Can satisfy:** depends on the issuer's schema. Typically age, country, OFAC, and proof of human.
* **UX:** the user completes a KYC flow once with the partner; later verifications reuse the attestation.
* **Notes:** see the [KYC spec](../reference/document-specifications/kyc.md) for the attestation format.

## Which document satisfies which rule

| Rule | Biometric passport | Aadhaar | KYC attestation |
| --- | :---: | :---: | :---: |
| Minimum age | ✅ | ✅ | ✅ (issuer-dependent) |
| Excluded countries | ✅ | ❌ | ✅ (issuer-dependent) |
| OFAC | ✅ | ❌ | ✅ (issuer-dependent) |
| Proof of human | ✅ | ✅ | ✅ (issuer-attested) |

If a user's document can't satisfy a rule your flow requires (for example Aadhaar against a country rule), that verification can't pass, the user would need a document that does.

## Related

* [Anatomy of a flow](anatomy.md).
* [Disclosures](disclosures.md).
* [Supported countries](../reference/supported-countries.md).
* [Aadhaar spec](../reference/document-specifications/aadhaar.md).
* [KYC spec](../reference/document-specifications/kyc.md).
