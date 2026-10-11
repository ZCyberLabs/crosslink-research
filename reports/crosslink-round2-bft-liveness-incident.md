# Crosslink Round 2 — Tenderlink/BFT Liveness Incident Observation

**Publisher:** ZCyberLabs  
**Status:** Incident observation; root cause unresolved  
**Scope:** Participating Crosslink feature-net finalizer  
**Disclosure level:** Public / sanitized

## Executive summary

After Tenderlink/BFT activity became live on the participating finalizer, the node successfully completed an earlier BFT height and then stopped making local BFT progress at the following height while the underlying network continued advancing.

The retained logs also show extreme STP receive-side queue pressure and long processing delays during the incident window. The receive-side pressure coincided with the liveness failure and may have contributed to it. The exact causal relationship remains unresolved, and the evidence is not sufficient to identify the implementation-level root cause.

Accordingly, ZCyberLabs is publishing this as a **liveness incident observation**, not as a confirmed Crosslink/Tenderlink bug attribution.

## Direct observations

### BFT progress

- BFT Height 3 reached a decision.
- The local finalizer then entered Height 4 / Round 0.
- Height 4 progressed from proposal handling into prevote.
- At `11:48:51 UTC`, the local finalizer emitted a NIL prevote for Height 4.
- The local node did not subsequently demonstrate a successful Height 4 decision in the retained incident window.
- The underlying chain/network continued progressing while local BFT progress remained stuck.

### Vote/proposal convergence

The retained incident analysis did not show a locally observed same-value quorum sufficient for decision at the stalled height.

Logs also contained vote/proposal divergence indicators rather than clean convergence on one locally actionable value. A full hash-by-hash reconstruction is still being prepared; this report therefore does not claim proposer equivocation or identify a faulty validator.

### STP receive pressure

During the incident, the STP receive path reached severe pressure:

- queue size reached `67,108,864` bytes
- approximately `84–85k` messages accumulated
- drops were observed at a rate on the order of thousands per 10-second interval
- one processing interval handled `34,179` messages over `192.267 s`
- a later interval handled `84,782` messages over `510.457 s`

These figures are direct evidence of sustained receive-side overload/backpressure during the same broad failure period.

## Current interpretation

### Strongly supported

The finalizer was not simply offline. It had participated successfully in BFT, entered the next height/round, and continued receiving substantial network traffic while local BFT progress failed.

The magnitude and duration of STP receive pressure are consistent with local Tenderlink timers or event processing being delayed enough to affect liveness.

### Inferred, not proven

A plausible working hypothesis is that STP receive-load/backpressure contributed to timer or event-loop starvation, which in turn prevented timely consensus progress.

This remains an inference. The retained evidence does **not yet prove**:

- the exact source-code defect
- that STP pressure was the only cause
- proposer equivocation
- that any particular validator caused the stall
- that independent reports from other operators have the same root cause

## Why no automatic restart was used as the first response

The incident state was treated as evidence. Restarting immediately could have destroyed information needed to distinguish a transient recovery symptom from the underlying liveness failure.

The preferred incident procedure is therefore:

1. preserve height/round/proposal/vote state and receive-pressure evidence;
2. record process/service state;
3. only then perform a controlled restart if recovery testing is required;
4. compare pre- and post-restart BFT progress.

## Evidence retained privately

The private incident bundle includes relevant journal/log windows and service-state captures. Public release intentionally omits host identifiers, access details, secret material, and operator identity mappings.

A more detailed message-level reconstruction can be shared privately with the relevant engineering team.

## Open questions

- Can the Height 4 event be reproduced under controlled receive load?
- Do proposal/value hashes show deterministic divergence across multiple finalizers?
- Does a controlled restart recover BFT progress without any other state change?
- Is the receive pressure causal, amplifying, or merely correlated?
- Which code path is responsible for the long receive-processing intervals?

## Disclosure posture

This report intentionally separates **observation** from **root-cause attribution**. Any future confirmed bug statement should be backed by reproducible evidence or source-level diagnosis.
