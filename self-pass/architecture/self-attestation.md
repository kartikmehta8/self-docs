# Verification Result

The `VerificationResult` type is the response returned by `SelfBackendVerifier.verify()` from `@selfxyz/core`. It contains the verification status, disclosed identity attributes, and user context data.

## VerificationResult

```typescript
type VerificationResult = {
  attestationId: AttestationId;
  isValidDetails: {
    isValid: boolean;
    isMinimumAgeValid: boolean;
    isOfacValid: boolean;
  };
  forbiddenCountriesList: string[];
  discloseOutput: GenericDiscloseOutput;
  userData: {
    userIdentifier: string;
    userDefinedData: string;
  };
};
```

### Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `attestationId` | `AttestationId` | Identifier for the attestation type (passport, national ID, etc.) |
| `isValidDetails.isValid` | `boolean` | Whether the ZK proof verified successfully on-chain |
| `isValidDetails.isMinimumAgeValid` | `boolean` | Whether the user meets the configured minimum age requirement |
| `isValidDetails.isOfacValid` | `boolean` | Whether the user passed the OFAC sanctions check |
| `forbiddenCountriesList` | `string[]` | Country codes unpacked from the proof's public signals |
| `discloseOutput` | `GenericDiscloseOutput` | Disclosed identity attributes from the ZK circuit |
| `userData.userIdentifier` | `string` | User identifier extracted from the verification context |
| `userData.userDefinedData` | `string` | Application-defined data passed through the verification flow |

## GenericDiscloseOutput

The disclosed identity attributes requested during verification. Only fields that were requested in the disclosure configuration will contain meaningful values.

```typescript
type GenericDiscloseOutput = {
  nullifier: string;
  forbiddenCountriesListPacked: string[];
  issuingState: string;
  name: string;
  idNumber: string;
  nationality: string;
  dateOfBirth: string;
  gender: string;
  expiryDate: string;
  minimumAge: string;
  ofac: boolean[];
};
```

### Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `nullifier` | `string` | Unique nullifier derived from the proof (prevents double-use) |
| `forbiddenCountriesListPacked` | `string[]` | Packed representation of forbidden countries from the circuit |
| `issuingState` | `string` | Three-letter code of the document's issuing country |
| `name` | `string` | Full name from the identity document |
| `idNumber` | `string` | Document number |
| `nationality` | `string` | Three-letter nationality code |
| `dateOfBirth` | `string` | Date of birth |
| `gender` | `string` | Gender as recorded on the document |
| `expiryDate` | `string` | Document expiry date |
| `minimumAge` | `string` | The verified minimum age value |
| `ofac` | `boolean[]` | OFAC check results |

## VerificationConfig

The configuration passed to `SelfBackendVerifier` to specify what checks to perform.

```typescript
type VerificationConfig = {
  minimumAge?: number;
  excludedCountries?: Country3LetterCode[];
  ofac?: boolean;
};
```

| Field | Type | Description |
| ----- | ---- | ----------- |
| `minimumAge` | `number` (optional) | Minimum age requirement. Verification fails if the user is younger. |
| `excludedCountries` | `Country3LetterCode[]` (optional) | List of three-letter country codes to exclude |
| `ofac` | `boolean` (optional) | Whether to check the user against the OFAC sanctions list |
