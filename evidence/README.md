# Evidence Index

This public directory is intentionally minimal.

The complete raw evidence is retained privately. Public artifacts must never contain:

- seed material or private keys
- credentials or access tokens
- SSH details
- private host/IP information that is not necessary to reproduce a finding
- personal/operator identity mappings
- unredacted configuration values that expose secrets

## Retained evidence classes

1. Effective systemd service configuration and process identity
2. Seed-guard helper metadata and positive/negative validation output
3. Restart and real-reboot recovery captures
4. Crash/core-handling checks
5. Tenderlink/BFT journal windows around the Height 3 -> Height 4 transition
6. STP queue/backpressure and drop observations
7. Hashes of relevant binaries/helpers for internal provenance

Raw evidence can be provided privately to the relevant Crosslink engineering team when needed.
