# Netscan Cutover Gate Checklist and Rollback Protocol (M4-T3)

## Purpose

Define the minimum gates required before any netscan-to-Bastion cutover decision and provide an operator-safe rollback protocol.

## Cutover Gate Checklist

All gates must be marked PASS before cutover.

| Gate | Requirement | Evidence | Status |
| --- | --- | --- | --- |
| G1 | M1 control-plane contracts complete | IPC schema + malformed/fuzz tests + rollback-safe reload | PASS |
| G2 | M2 health/telemetry contracts complete | Health snapshot schema, counter consistency, scanner latency bucket tests | PASS |
| G3 | M3 fault tolerance contracts complete | Worker crash recovery, IPC backpressure, timeout/retry tests | PASS |
| G4 | M4 parity artifacts complete | Parity matrix + fixture-backed overlapping output tests | PASS |
| G5 | Release candidate local validation | Targeted deterministic suites run and documented | PASS |
| G6 | Operator runbook references current | Security, privacy, local execution, release docs aligned | PASS |

## Cutover Decision Rule

Proceed only if G1-G6 are all PASS for the same release candidate branch tip.

## Rollback Triggers

Immediately rollback if any of the following occur post-cutover:

1. IPC response validation errors in operator workflows.
2. Health snapshot schema mismatch or parser failure in downstream systems.
3. Regression in parity-critical discovery envelope fields.
4. Sustained degraded worker state due to restart budget exhaustion.
5. Operator inability to complete required discovery/report workflow.

## Rollback Protocol

1. Freeze changes:

- Stop introducing new parity-affecting changes until rollback stabilizes.

1. Revert deployment pointer:

- Redeploy previous known-good release artifact.
- Restore previous service configuration and startup command set.

1. Validate daemon baseline:

- Confirm daemon startup, socket readiness, and ping/status command paths.
- Verify no malformed IPC loops and no timeout storm behavior.

1. Validate reporting baseline:

- Generate discovery JSON output and confirm envelope shape.
- Run fixture parity check suite for overlapping fields.

1. Confirm operator functionality:

- Execute required operator runbook path end-to-end.
- Confirm reports are ingestible by downstream consumers.

1. Incident record:

- Record failure mode, trigger timestamp, and rollback completion timestamp.
- Link failing commit range and required remediation owner.

## Minimum Post-Rollback Exit Criteria

1. Baseline daemon tests pass locally.
2. Parity fixture tests pass locally.
3. Health/status contracts pass locally.
4. Root cause is documented with remediation milestone assignment.

## Ownership and Sign-off

- Architecture owner: parity and cutover gate approval.
- Daemon owner: IPC, lifecycle, and worker resilience sign-off.
- Utils/report owner: report fixture parity sign-off.
- Operations owner: rollback rehearsal and runbook sign-off.
