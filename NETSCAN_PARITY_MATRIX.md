# Netscan Parity Matrix (M4-T1)

## Objective

Define the first explicit capability map for netscan replacement planning: what Bastion already replaces, what remains deferred, and what is intentionally dropped from parity scope.

## Scope Legend

- REPLACE: Bastion has equivalent or stronger capability now.
- PARTIAL: Bastion has partial parity; remaining work is identified.
- DEFER: Capability exists in concept/roadmap but not ready for cutover.
- DROP: Capability is intentionally not carried forward as-is.

## Capability Matrix

| Legacy netscan capability | Bastion target capability | Current status | Replacement path | Notes |
| --- | --- | --- | --- | --- |
| IPv4/IPv6 blocklist controls via CLI | Daemon IPC rule controls (`block`/`unblock`) | REPLACE | Keep Bastion CLI + daemon IPC as source of truth | Versioned IPC + schema contracts already in place |
| Layer 4 deny policy (TCP/UDP) | Rule engine Layer 4 block/unblock + counters | REPLACE | Maintain in core + daemon status health snapshot | Counter consistency contracts completed |
| Stateless recon summary counters | `ReconStatelessScan` telemetry record + response envelope | REPLACE | Keep typed recon telemetry schema in daemon protocol | Contract-tested response shape |
| Scanner latency visibility | Health snapshot scanner latency buckets | REPLACE | Runtime telemetry now records scan latency observations and serves live buckets in status snapshots | Contract-tested runtime bucket classification + daemon suite coverage |
| Queue pressure visibility | Health snapshot queue pressure fields | REPLACE | Runtime telemetry now tracks live queue depth and high-watermark and serves them in status snapshots | Contract-tested queue pressure behavior + daemon suite coverage |
| Worker crash resilience | Worker crash supervisor restart/escalation contracts | REPLACE | Keep bounded restart policy and expose status in health path | M3-T1 contracts complete |
| IPC malformed payload resilience | Framing guards + schema validation + fuzz/backpressure contracts | REPLACE | Continue bounded framing + validation strategy | M1 and M3 IPC hardening suites complete |
| Request timeout and retry lifecycle | Timeout-aware framing helper + retry contracts | REPLACE | Keep bounded retry policy at daemon/CLI boundaries | M3-T3 contracts complete |
| OUI/vendor lookup (legacy CSV style) | Dynamic resolver model in Bastion utilities | DEFER | Complete dynamic update + cache policy and promote into cutover gate | Not a blocker for daemon control-plane cutover |
| Legacy monolithic scanner UX paths | Bastion modular recon/daemon workflows | DROP | Preserve only proven operator flows; avoid carrying brittle monolith paths | Intentional architecture divergence |

## Initial Cutover Gates (Draft)

1. REPLACE rows must have passing deterministic contract tests in current release candidate.
2. PARTIAL rows must include explicit risk notes and operator-visible degraded behavior.
3. DEFER rows must have backlog milestone mapping before any netscan decommission decision.
4. DROP rows require architecture sign-off and migration guidance for operators.

## Risk Register (Initial)

- Scanner latency and queue pressure currently rely on placeholder runtime values in daemon health snapshots.
- Final cutover requires live runtime wiring so status reflects observed production behavior.
- Legacy scripts still exist in parallel; migration sequencing must avoid operator confusion.

## Next Artifacts

1. Fixture-based parity validation tests for overlapping output paths.
2. Cutover checklist and rollback protocol with operator runbook links.
3. Explicit ownership review and sign-off for DEFER and DROP rows.
