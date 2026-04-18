# Checkpoint 1 Draft

Due: Monday, April 20, 2026, in class.

## Project

Option 2: Design of a Software License Server Immune to Software and Microarchitectural Attacks.

## Baseline Project Claim

We will build and evaluate a portable software-first license enforcement subsystem. The checker is implemented as a Rust license core statically linked into a C toy host application. The host application enters its protected path only after the Rust core accepts a valid signed license.

The baseline does not require TPM, TEE, measured boot, OS keychain, or special hardware support. Those mechanisms are treated as enhanced deployment options for reducing residual risks such as binary patching or privileged compromise.

## Security Goal

The protected application should execute only when a valid license is present. The design should reject forged, malformed, expired, wrong-product, ambiguous, or tampered licenses and should fail closed on parser, verifier, policy, or FFI errors. It should also minimize microarchitectural leakage risk by avoiding client-side private signing secrets and reducing secret-dependent behavior.

## Threat Model Summary

In scope:

- Level 1 malicious input attacker who controls license files and serialized inputs
- Level 2 local user-space attacker who can run the binary, fuzz inputs, modify local files, observe timing/cache behavior, and attempt memory-corruption or control-flow attacks

Out of scope for the baseline:

- compromised issuer signing key
- malicious kernel, hypervisor, firmware, or microcode
- invasive physical attacks
- complete OS loader or code-signing compromise
- denial of service

Key residual risk: if an attacker can freely patch and run a modified binary without any platform code-integrity mechanism, a software-only checker cannot absolutely prevent bypass. Enhanced deployments such as OS code signing, measured boot, TPM, or TEE support are discussed separately.

## Architecture Summary

```text
Untrusted license file
        |
        v
C host entry wrapper
        |
        v
Rust C ABI boundary
        |
        v
Rust license core
  - canonical CBOR parser
  - Ed25519 verifier
  - policy engine
  - fail-closed decision logic
        |
        v
ALLOW or DENY
        |
        +--> DENY: exit before protected code
        |
        +--> ALLOW: enter protected application path
```

Implementation decisions:

- Rust license core for parser, verifier, policy, and decision logic
- C toy host for application integration
- static linking for the baseline
- Cargo plus Makefile build system
- canonical CBOR license format
- Ed25519 signatures using `ed25519-dalek`
- client stores only the Ed25519 public verification key
- C ABI exposes only allow or deny
- device binding deferred to optional future work

## License Fields

Minimum fields:

- `format_version`
- `license_id`
- `product_id`
- `features`
- `issued_at`
- `expires_at`
- `signature_algorithm`
- `signature`

Optional future fields:

- `device_binding`
- `customer_id`
- `license_tier`
- `max_application_version`
- `revocation_epoch`

## Project Plan

1. Finalize checkpoint materials: threat model, architecture, plan, artifact list, and collaboration plan.
2. Build minimal prototype: Rust static library, C host, C ABI, canonical CBOR parser, Ed25519 verifier, and policy checks.
3. Add correctness tests: valid, missing, malformed, tampered, expired, wrong-product, and invalid-signature licenses.
4. Add software attack evaluation: malformed corpus, parser stress/fuzzing, C ABI misuse tests, sanitizer builds for C host code.
5. Add microarchitectural evaluation: timing variance measurements, protected-data access review, simulated fault injection for corrupted license or decision state, and documented speculation/Rowhammer assumptions.
6. Prepare final report, result artifacts, demo material, and required Gemini logs.

## Expected Final Artifacts

- Rust license core source code
- C toy host source code
- C ABI header
- Makefile and Cargo configuration
- license-generation helper scripts
- unit tests and integration tests
- malformed and tampered license test corpus
- timing and fault-injection evaluation scripts
- raw and processed result outputs
- final report
- slides or demo material
- Gemini logs

## Collaboration Plan

Fill in team-specific details:

- meeting cadence:
- member responsibilities:
- code review process:
- documentation owner:
- implementation owner:
- evaluation owner:
- final report owner:
