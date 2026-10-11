# ZCyberLabs Crosslink Research

Independent security, reliability, and performance research around the Zcash Crosslink feature-net.

This repository is intended to publish concise, reproducible technical findings while keeping sensitive operational material out of the public record.

## Current reports

- `reports/crosslink-round2-finalizer-hardening.md` — hardened finalizer operating baseline and validation results.
- `reports/crosslink-round2-bft-liveness-incident.md` — observed Tenderlink/BFT liveness incident and current evidence boundaries.

## Evidence policy

Public reports contain sanitized evidence only. They intentionally exclude seed material, private keys, credentials, host access details, operator identity mappings, and other secrets. Full raw evidence is retained privately and can be shared with the relevant engineering team when needed for verification.

## Status vocabulary

- **Observed** — directly present in retained logs, service state, or test output.
- **Validated** — a deliberate positive or negative test produced the expected outcome.
- **Inferred** — a technically plausible explanation supported by observations, but not proven as root cause.
- **Open** — requires additional evidence or reproduction.

These reports are research notes, not a formal third-party security audit.
