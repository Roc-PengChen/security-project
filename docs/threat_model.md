# Threat Model

## Security Boundary

The baseline design is a portable software-first license enforcement system that runs on standard processors and operating systems available today. It does not require TPMs, TEEs, measured boot, OS keychains, or special hardware support for the first prototype.

Enhanced deployment modes, such as TPM-backed measurements, TEE isolation, OS code signing, or platform key storage, are treated as optional extensions and residual-risk mitigations rather than baseline requirements.

## Security Goal

The protected application must execute only after a valid signed license has been parsed, verified, and accepted by policy. Invalid, malformed, expired, forged, replayed, or wrong-product licenses must fail closed. Attackers should not be able to bypass the enforcement decision through software bugs, unsafe parsing, control-flow manipulation, or practical microarchitectural leakage from the license checker.

## Assets

- license issuer signing key: held only by the offline issuer and never embedded in the client
- client trust anchor: Ed25519 public verification key embedded in or provisioned to the checker
- canonical CBOR license payload: product id, feature set, issue time, expiration time, license id, and version
- license signature: Ed25519 signature over the canonical CBOR payload
- authorization decision: the allow/deny result returned by the Rust license core
- checker integrity: parser, verifier, policy engine, and C ABI boundary
- protected application entry path: the code path that should run only after an allow decision
- evaluation evidence: tests, fuzzing results, timing data, and fault-injection experiments used to justify the design

## In-Scope Attacker Model

### Level 1: Malicious Input Attacker

The attacker can provide arbitrary license inputs, including malformed CBOR, non-canonical encodings, duplicated or missing fields, oversized fields, tampered payloads, invalid signatures, expired licenses, and wrong-product licenses.

Expected result: the checker rejects all invalid inputs without crashing, accepting ambiguous encodings, or entering the protected application path.

### Level 2: Local User-Space Attacker

The attacker can run the protected binary locally, observe its behavior, repeatedly invoke it, modify local files, fuzz inputs, attach ordinary user-space tooling where the OS permits it, attempt memory-corruption and control-flow attacks, and observe timing or cache-level effects on shared hardware.

Expected result: the design minimizes trusted code, uses Rust for parser/verifier/policy logic, exposes only a small C ABI, fails closed on unexpected states, and avoids client-side private secrets whose leakage would permit license forgery.

Important limitation: if a local attacker can freely patch and execute a modified binary without any code-integrity mechanism, a purely software-only design cannot absolutely prevent bypassing the checker. This is documented as a residual risk and addressed in enhanced deployment options such as code signing, measured boot, TPM, or TEE support.

## Out-of-Scope Attacker Model

### Level 3: Privileged or Platform Attacker

The following are out of scope for the baseline prototype:

- compromised license issuer signing key
- malicious kernel, hypervisor, firmware, or microcode
- invasive physical attacks
- hardware debug access that can arbitrarily modify process execution
- complete compromise of the OS loader or code-signing policy
- denial-of-service attacks that only prevent the application from running

These attacks may be discussed as residual risks or enhanced-deployment motivations.

## Trust Assumptions

- Ed25519 remains unbroken for the project threat model.
- The license issuer protects the private signing key.
- The selected Rust crypto and CBOR libraries behave according to their documented security properties.
- The Rust license core is built from reviewed source with expected compiler settings.
- The C host calls the license core through the intended C ABI before entering the protected path.
- The baseline OS provides ordinary process isolation but is not trusted against privileged compromise.

## Security Properties

- Only Ed25519-signed canonical CBOR licenses from the trusted issuer can authorize execution.
- The signature covers the full canonical payload, excluding only the signature container itself.
- Non-canonical, ambiguous, malformed, duplicated, or oversized license data is rejected.
- Missing fields, unsupported versions, expired licenses, wrong product ids, and invalid features are rejected.
- All parser, verifier, FFI, and policy errors return deny.
- The C ABI exposes only a minimal allow/deny result to the host application.
- Detailed internal errors may be used for tests, but production-facing logs must not leak sensitive payload details.
- The client does not contain the issuer private key or any other long-term signing secret.
- Authorization state should be short lived and should not be stored as a long-lived mutable global flag.

## Attack-to-Mitigation Matrix

| Threat | Relevant Attacker | Mitigation | Validation |
| --- | --- | --- | --- |
| Forged license | Level 1 | Ed25519 signature over canonical CBOR payload; public key only on client | positive and negative signature tests |
| Tampered fields | Level 1 | signature covers product id, features, times, version, and license id | mutate each field and verify deny |
| Malformed parser input | Level 1 | strict canonical CBOR decoding, field bounds, duplicate rejection, fail closed | malformed corpus, fuzzing, unit tests |
| Ambiguous serialization | Level 1 | canonical CBOR requirement; reject non-canonical encodings | canonicalization tests |
| Expired or wrong-product license | Level 1 | policy engine checks time and product id after signature verification | policy tests |
| FFI misuse | Level 2 | minimal C ABI, null pointer and length checks, no cross-boundary ownership in prototype | C integration tests |
| Memory corruption in checker | Level 2 | Rust implementation for parser/verifier/policy; small unsafe boundary | Rust tests, sanitizer builds for C host |
| Control-flow bypass in host | Level 2 | narrow entry wrapper, single decision gate, fail-closed default | integration tests and code review |
| Timing or cache leakage | Level 2 | no client private key; avoid secret-dependent branching/table indexing; use crypto library routines | timing measurements and qualitative review |
| Speculative execution effects | Level 2 | avoid touching protected data before allow; optional architecture-specific fences at authorization boundary | architecture review and optional experiments |
| Rowhammer-style faults | Level 2 | avoid long-lived mutable authorization flag; recheck critical state; deny on inconsistent state; document platform dependence | simulated bit-flip/fault-injection tests |

## Residual Risks

- A software-only checker cannot fully prevent bypass if attackers can patch and run arbitrary modified binaries.
- Microarchitectural behavior varies across CPUs and cannot be universally proven safe by a portable prototype.
- Rowhammer resilience depends on DRAM, ECC, refresh policy, OS allocation behavior, and platform mitigations.
- Device binding is not part of the initial prototype because local device identifiers are often spoofable without stronger platform support.
- Enhanced deployment with TPM, TEE, measured boot, or OS code signing can reduce some residual risks but is outside the baseline implementation.
