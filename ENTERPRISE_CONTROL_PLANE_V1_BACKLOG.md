# Bastion Enterprise Control Plane v1 Backlog

## Objective

Build the reliability, policy, observability, and governance layers that move Bastion from strong scanner capabilities into a production-grade enterprise platform and replacement track for netscan.

## North Star Outcomes

1. Bastion daemon and clients operate safely under malformed input, partial failure, and privilege-constrained environments.
2. Policy/rule lifecycle is atomic, auditable, and rollback-safe.
3. Operators get deterministic health visibility and explicit degraded-mode behavior.
4. Netscan parity and cutover readiness are measurable with hard acceptance gates.

## Execution Principle

TDD-first, then implementation. No milestone is complete until both deterministic tests and operational runbooks are green.

## Milestone Plan

### M1: Contract and Policy Safety

Scope:

1. Version daemon protocol and enforce strict request/response schema validation.
2. Add atomic rule persistence and rollback on failed reload.
3. Enforce policy update idempotency and sequence safety.

TDD inventory to add first:

1. daemon/tests/ipc_schema_contract_tests.rs
2. daemon/tests/ipc_malformed_payload_fuzz_tests.rs
3. core/tests/rule_state_atomic_reload_tests.rs
4. core/tests/rule_persistence_roundtrip_tests.rs

Definition of done:

1. Malformed IPC payloads never panic the daemon and return structured errors.
2. Failed reload leaves previous active rules intact.
3. Re-applying same policy is idempotent and side-effect stable.

### M2: Runtime Health and Telemetry

Scope:

1. Publish health snapshot endpoint through daemon status path.
2. Expose counters: active rules, drops by protocol, scanner latency buckets, queue pressure.
3. Add deterministic degraded-state flags (no privilege, no adapter capability, fallback mode).

TDD inventory to add first:

1. daemon/tests/health_snapshot_contract_tests.rs
2. core/tests/counter_consistency_tests.rs
3. utils/tests/scanner_latency_metrics_tests.rs

Definition of done:

1. Health snapshot schema is versioned and contract-tested.
2. Counter values remain monotonic and internally consistent.
3. Degraded states are explicit and documented, never implicit.

### M3: Fault Tolerance and Recovery

Scope:

1. Add worker/child crash recovery tests for daemon orchestration paths.
2. Validate backpressure handling under burst load.
3. Add timeout and retry contracts for IPC request lifecycle.

TDD inventory to add first:

1. daemon/tests/worker_crash_recovery_tests.rs
2. daemon/tests/ipc_backpressure_tests.rs
3. daemon/tests/request_timeout_retry_tests.rs

Definition of done:

1. Forced worker failure does not take down daemon control plane.
2. Backpressure does not cause unbounded memory growth.
3. Request timeout behavior is deterministic and surfaced to operators.

### M4: Netscan Replacement Readiness

Scope:

1. Create capability parity matrix (replace, defer, intentionally drop).
2. Add side-by-side fixture verification for overlapping outputs.
3. Publish cutover gate checklist and rollback protocol.

TDD inventory to add first:

1. utils/tests/netscan_parity_fixture_tests.rs
2. docs/tests/replacement_gate_doc_consistency.md (checklist artifact)

Definition of done:

1. Parity matrix is complete and reviewed.
2. Shared output contracts pass against fixtures.
3. Cutover and rollback steps are documented and rehearsed.

## Priority Queue (Start Here)

1. M1-T1: daemon protocol version field and schema validator.
2. M1-T2: atomic rule reload path with rollback guard.
3. M2-T1: health snapshot endpoint and status contract tests.
4. M3-T1: worker crash recovery test harness.
5. M4-T1: initial netscan parity matrix document.

## Initial Test Command Set

1. ./test.sh --ipc
2. ./test.sh --core
3. ./test.sh --health
4. ./test.sh --worker
5. ./test.sh -p utils netscan_parity_fixture_tests -- --nocapture

## Release Gate for Enterprise Control Plane v1

All must be true:

1. M1 through M4 definition-of-done gates are satisfied.
2. Workspace tests are green on release candidate.
3. Manual live smoke workflows pass with explicit targets only.
4. Operational runbook references are up to date.

## Suggested Ownership Split

1. Core and daemon safety contracts: core and daemon track.
2. Scanner and telemetry metrics contracts: utils track.
3. Cutover matrix and readiness docs: architecture and operations track.

## Current Status

M1 completed and validated.
M2-T1 completed and locally validated.
M2-T2 completed and locally validated.
M2-T3 completed and locally validated.
M3-T1 completed and locally validated.
M3-T2 completed and locally validated.
M3-T3 completed and locally validated.
M4-T1 completed and reviewed locally.
M4-T2 completed and locally validated.
M4-T3 completed and reviewed locally.
M4-T4 completed and locally validated.
M4-T5 completed and locally validated.

### M1 Completion Evidence

1. Versioned IPC envelopes with schema validation are active in CLI and daemon.
2. Malformed/oversized/truncated payload handling is contract-tested.
3. Atomic reload rollback on apply failure is implemented and tested.
4. Ruleset persistence roundtrip and guarded reload contracts are implemented and tested.

### M2-T1 Completion Evidence

1. `status` path now returns a versioned health snapshot payload (`HEALTH_SNAPSHOT_SCHEMA_VERSION`).
2. Health snapshot schema includes active rules, drop counters, latency buckets, queue pressure, and explicit degraded-state flags.
3. New daemon contract test suite validates health snapshot acceptance and malformed shape rejection.
4. Daemon and CLI crates pass local test execution after status-path migration.

### M2-T2 Completion Evidence

1. Added core counter snapshot API (`CounterSnapshot`) with deterministic aggregate total calculation.
2. Added monotonicity and consistency test coverage in `core/tests/counter_consistency_tests.rs`.
3. Verified cross-protocol counter isolation (IPv4 vs IPv6 vs Layer 4) and concurrent increment integrity.
4. Full core crate test suite passes locally including the new consistency suite.

### M2-T3 Completion Evidence

1. Added reusable scanner latency histogram utility in `utils::net::scanner_metrics`.
2. Added deterministic bucket boundaries (`<=5ms`, `<=20ms`, `<=100ms`, `>100ms`) with total sample accounting.
3. Added integration contract tests in `utils/tests/scanner_latency_metrics_tests.rs` for boundary, monotonicity, and total consistency checks.
4. New latency metrics tests pass locally, plus a representative existing scanner test regression check.

### M3-T1 Completion Evidence

1. Added a bounded worker crash supervisor state machine in `daemon/src/lifecycle/worker_supervisor.rs`.
2. Implemented deterministic recovery outcomes for restart, stale-report ignore, and escalation on restart budget exhaustion.
3. Added daemon crash-recovery contract tests in `daemon/tests/worker_crash_recovery_tests.rs`.
4. New worker recovery tests and full daemon crate test suite pass locally.

### M3-T2 Completion Evidence

1. Added IPC burst/backpressure contract tests in `daemon/tests/ipc_backpressure_tests.rs`.
2. Added symmetric payload-size guard on IPC write path in `daemon/src/ipc/framing.rs`.
3. Validated that oversized writes are rejected before transmission and do not poison subsequent valid messages.
4. New backpressure suite and full daemon crate tests pass locally.

### M3-T3 Completion Evidence

1. Added deterministic timeout framing helper `read_message_with_timeout` in `daemon/src/ipc/framing.rs`.
2. Added request lifecycle timeout/retry contracts in `daemon/tests/request_timeout_retry_tests.rs`.
3. Validated retry success for late-arriving payloads within bounded attempts.
4. Validated deterministic timeout error behavior and stop-after-budget semantics.

### M4-T1 Completion Evidence

1. Added initial parity matrix artifact: `docs/NETSCAN_PARITY_MATRIX.md`.
2. Classified replacement scope into `REPLACE`, `PARTIAL`, `DEFER`, and `DROP` lanes.
3. Captured initial cutover gate criteria and risk register for operator review.
4. Documented next artifacts required for full M4 completion (fixtures + cutover rollback checklist).

### M4-T2 Completion Evidence

1. Added fixture-backed parity test suite: `utils/tests/netscan_parity_fixture_tests.rs`.
2. Added snapshot fixture for discovery CSV overlap fields: `utils/tests/fixtures/netscan/discovery_csv_expected.csv`.
3. Validated side-by-side parity for overlapping JSON envelope/device fields against RMEDIAtech fixtures.
4. New parity test suite passes locally (`./test.sh --parity`).

### M4-T3 Completion Evidence

1. Added cutover gate + rollback artifact: `docs/NETSCAN_CUTOVER_GATE_CHECKLIST.md`.
2. Defined explicit cutover decision rule and rollback trigger set.
3. Added deterministic rollback protocol and post-rollback exit criteria.
4. Captured ownership/sign-off lanes for architecture, daemon, utils/report, and operations.

### M4-T4 Completion Evidence

1. Added doc consistency contract test suite: `utils/tests/replacement_gate_doc_consistency_tests.rs`.
2. Validated parity matrix required sections and cutover gate markers are present.
3. Validated backlog references for M4 artifacts remain anchored.
4. New doc consistency suite passes locally (`./test.sh --consistency`).

### M4-T5 Completion Evidence

1. Wired runtime telemetry collector for daemon health snapshots in `daemon/src/observability/runtime_telemetry.rs`.
2. Health snapshot now reports live scanner latency buckets and queue depth/high-watermark instead of static placeholders.
3. Added telemetry contract tests in `daemon/tests/runtime_health_telemetry_tests.rs`.
4. New telemetry tests and full daemon crate test suite pass locally.
