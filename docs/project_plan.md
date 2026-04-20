# Project Plan

## Milestone 1: Checkpoint 1 Package

Target date: Monday, April 20, 2026.

- finalize portable software-first security boundary
- document enhanced deployment options separately from the baseline
- define Level 1 and Level 2 attackers as in scope
- define Level 3 privileged/platform attackers as out of scope
- specify Rust license core plus C toy host integration
- specify canonical CBOR license format
- specify Ed25519 signatures using `ed25519-dalek`
- specify static linking through Cargo plus Makefile
- define prototype scope and final artifact list

## Milestone 2: Minimal Correctness Prototype (4/21/2026 - 4/26/2026)
- create Rust crate under `src/license_core`
- create C toy host under `src/app_integration`
- create C ABI header under `include`
- build Rust license core as a static library
- link the static library into the C host with a Makefile
- implement canonical CBOR license decoding
- implement Ed25519 signature verification
- implement product id, feature, issue time, expiration time, license id, and version policy checks
- implement a license-generation helper for tests
- add Rust unit tests for parser, verifier, and policy behavior
- add integration tests showing valid licenses enter the protected path and invalid licenses do not

## Milestone 3: Software Attack Evaluation (4/27/2026 - 5/2/2026)

- build malformed CBOR and tampered-license test corpus
- reject duplicated fields, missing fields, non-canonical encodings, oversized fields, and unsupported versions
- mutate each signed field and verify the checker denies
- test null pointer, unreadable path, invalid UTF-8, and oversized input behavior at the C ABI boundary
- run sanitizer builds for the C toy host and integration layer where available
- run fuzzing or parser stress tests against the Rust parser
- document how fail-closed behavior is enforced in the API and implementation

## Milestone 4: Microarchitectural Risk Evaluation (5/3/2026 - 5/9/2026)

- identify whether the checker contains license-dependent branches or memory accesses that affect sensitive state
- measure valid-license versus invalid-license timing variance for the prototype
- optionally measure cache-observable behavior for selected paths if tooling and time permit
- evaluate whether protected data is accessed only after the allow decision
- simulate corrupted license bytes and corrupted decision state to model fault effects
- document speculative execution and Rowhammer assumptions instead of claiming universal immunity

## Milestone 5: Final Report and Demo (5/9/2026 - 5/12/2026)

- collect reproducible build and test commands
- save raw outputs under `results/raw`
- summarize processed results under `results/processed`
- write the final report with security goal, threat model, architecture, implementation, evaluation, limitations, and contributions
- prepare slides or a demo script
- include all required Gemini logs under `logs/gemini`

## First Prototype Success Criteria

- `make test` builds the Rust static library, builds the C host, and runs Rust plus integration tests
- a valid signed canonical CBOR license allows the toy host to enter `protected_main`
- missing, malformed, tampered, expired, wrong-product, or invalid-signature licenses deny
- C ABI misuse cases deny instead of crashing
- the checker contains no issuer private key or long-term client signing secret
- the documentation describes residual risks honestly
