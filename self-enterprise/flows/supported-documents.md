# Supported documents

Each credential type supports a different subset of rules and a different geographic footprint. Pick documents based on the user populations you serve.

## Biometric passport (ICAO 9303 e-passport)

**Highest assurance.** A passport-chip read produces a credential signed by the issuing state.

* **Coverage:** 60+ countries. See [Supported countries](../reference/supported-countries.md).
* **Supports:** every rule, age, country, OFAC, and proof of human.
* **UX:** the user holds the passport to the phone's NFC reader. About 30 seconds end to end.
* **Notes:** needs an NFC-capable phone (most iPhones and Android 8+).

## Aadhaar (India)

India's universal government ID.

* **Coverage:** India only.
* **Supports:** age and proof of human. Does **not** support country rules (Aadhaar does not encode nationality the same way) or OFAC directly.
* **UX:** the user scans the QR code on their Aadhaar and provides their share code.
* **Notes:** see the [Aadhaar spec](../reference/document-specifications/aadhaar.md) for the cryptographic detail.

## KYC attestation (partner-issued)

A credential issued by a Self partner KYC provider after a remote KYC check.

* **Coverage:** depends on the issuer. Most partners cover the US, EU, UK, and a long tail.
* **Supports:** depends on the issuer's schema. Typically age, country, OFAC, and proof of human.
* **UX:** the user completes a KYC flow once with the partner; later verifications reuse the attestation.
* **Notes:** see the [KYC spec](../reference/document-specifications/kyc.md) for the attestation format.

## Picking documents for your flow

* **Global consumer app:** biometric passport plus KYC attestation. Maximizes coverage.
* **India-only product:** Aadhaar plus biometric passport. Aadhaar penetration is far higher than passport ownership in India.
* **Compliance-heavy use case:** biometric passport only. Highest assurance, government-signed.
* **Reusable verification (verify once, trust it later):** KYC attestation, so the user does not rescan a document each time.

## Capability matrix

| Rule | Biometric passport | Aadhaar | KYC attestation |
| --- | :---: | :---: | :---: |
| Age (minimum / maximum) | ✅ | ✅ | ✅ (issuer-dependent) |
| Country (include / exclude) | ✅ | ❌ | ✅ (issuer-dependent) |
| OFAC | ✅ | ❌ | ✅ (issuer-dependent) |
| Proof of human | ✅ | ✅ | ✅ (issuer-attested) |

The Configure tab flags incompatible combinations. If you allow only Aadhaar and add a country rule, you will see a warning before publishing.

## Related

* [Anatomy of a flow](anatomy.md).
* [Disclosures](disclosures.md).
* [Supported countries](../reference/supported-countries.md).
* [Aadhaar spec](../reference/document-specifications/aadhaar.md).
* [KYC spec](../reference/document-specifications/kyc.md).
