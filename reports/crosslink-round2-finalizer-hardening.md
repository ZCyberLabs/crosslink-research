# Crosslink Round 2 — Finalizer Hardening & Reliability Baseline

**Publisher:** ZCyberLabs  
**Status:** Evidence-backed operating baseline  
**Scope:** Participating Crosslink feature-net finalizer  
**Disclosure level:** Public / sanitized

## Executive summary

During Crosslink Round 2, ZCyberLabs hardened and validated a participating finalizer beyond the default service configuration. The work focused on least privilege, secret handling, deterministic startup checks, restart/reboot recovery, runtime isolation, and crash-handling behavior.

The resulting baseline was frozen before further BFT-specific experimentation so that later consensus observations could be compared against a stable operational reference.

This document reports only controls and tests supported by retained evidence. It is not a claim that the node, Crosslink, Tenderlink, Zebra, or the host is free of vulnerabilities.

## Baseline controls

### 1. Unprivileged service execution — VALIDATED

The finalizer service was migrated from privileged execution to a dedicated service identity:

- `User=crosslink`
- `Group=crosslink`
- `UMask=0077`

The effective service process was verified to run under the dedicated account rather than root.

### 2. systemd sandboxing — VALIDATED

The frozen effective service configuration included:

- `NoNewPrivileges=yes`
- empty capability bounding and ambient capability sets
- `ProtectSystem=full`
- `ProtectHome=yes`
- `PrivateTmp=yes`
- `PrivateDevices=yes`
- kernel/control-group/hostname protections enabled
- `RestrictNamespaces=yes`
- `RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6 AF_NETLINK`
- `SystemCallArchitectures=native`
- `LockPersonality=yes`
- `RestrictRealtime=yes`
- `RestrictSUIDSGID=yes`

`AF_NETLINK` was retained because testing showed it was required for correct operation; removing it was therefore not treated as a valid hardening improvement.

### 3. Runtime and persistent-state separation — VALIDATED

Runtime state was moved out of privileged home-directory locations.

The frozen service uses:

- `RuntimeDirectory=crosslink`
- runtime directory mode `0700`
- persistent application state/cache under a dedicated `/var/lib/crosslink` hierarchy

This allows `ProtectHome=yes` without requiring the daemon to access `/root`.

### 4. Fail-closed seed guard — VALIDATED

A root-owned, non-user-writable pre-start guard was integrated with systemd via `ExecStartPre`.

Before the finalizer process is allowed to start, the guard verifies the configured seed file against a pinned expected state, including:

- regular file
- not a symbolic link
- exact expected byte length
- restrictive mode (`0600`)
- expected service ownership
- exact cryptographic digest match
- execution under the intended service identity

The digest itself and seed material are intentionally omitted from this public report.

Negative tests were performed for invalid conditions including missing file, invalid size, digest mismatch, symlink substitution, incorrect mode, incorrect ownership, and invalid execution context. The guard failed closed as intended.

A subsequent positive recovery test confirmed that restoring the expected seed state allowed normal service startup.

### 5. Restart policy — VALIDATED

The frozen effective service policy included:

- `Restart=on-failure`
- restart delay of 10 seconds
- bounded start-rate limiting

Restart behavior was exercised and the finalizer resumed operation without changing its finalizer identity.

### 6. Real host reboot recovery — VALIDATED

A real VPS reboot was performed after the hardening changes.

Post-reboot validation confirmed:

- the hardened service returned under the dedicated service identity
- runtime-directory creation was restored automatically
- seed protections remained effective
- persistent state/cache remained accessible at the intended location
- synchronization resumed
- finalizer identity remained unchanged

This test demonstrates reboot recovery of this configuration; it is not a high-availability claim.

### 7. Crash/core handling — VALIDATED

Crash-handling behavior was reviewed and hardened so that application crashes do not casually leave sensitive diagnostic artifacts available through the default host crash pipeline.

Apport was disabled/inactive in the frozen baseline, and core/crash behavior was checked as part of the operational hardening work.

## Evidence retained privately

ZCyberLabs retains the underlying evidence set, including effective unit configuration, service-state captures, positive/negative seed-guard test output, reboot/recovery observations, hashes of relevant binaries/helpers, and selected logs.

Public release intentionally excludes:

- seed contents
- private keys
- credentials
- operator/account identity mappings
- host access details
- values that unnecessarily fingerprint sensitive secret material

Evidence can be shared privately with the relevant engineering team when required for verification.

## Limitations

This baseline proves that the listed hardening controls operated as tested on the participating finalizer. It does not establish:

- complete host compromise resistance
- formal verification of the Crosslink/Tenderlink implementation
- Byzantine fault tolerance beyond protocol assumptions
- high availability
- absence of application-level consensus or networking bugs

Consensus behavior observed after BFT activation is documented separately.
