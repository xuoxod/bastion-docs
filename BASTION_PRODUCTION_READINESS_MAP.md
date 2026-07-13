# Bastion Production Readiness Map

## Scope

This document completes readiness across three tracks:

1. Track 1: production-ready now
2. Track 2: near-ready with bounded effort
3. Track 3: roadmap-grade or hardware-conditional capabilities

Companion planning and handoff material:

- docs/SAVVY_OPERATOR_RELEASE_HANDOFF_PLAN.md
- docs/LOCAL_EXECUTION_GUIDE.md
- docs/LAB_HOST_POLICY_MATRIX.md

## Executive Status

- Track 1: Ready to operate and release for operator-first workflows
- Track 2: Partially complete, medium effort to close
- Track 3: Planned, higher effort and environment-dependent

## Track 1: Production-Ready Now

### A. Core Engine Integrity

- Status: Ready
- Evidence anchors:
  - core/src/radix.rs
  - core/src/state.rs
  - core/tests/layer4_counters_tests.rs
- Why ready:
  - Lock-free path and radix behaviors are covered by unit and integration tests.
  - Layer4 counters and blocking behavior have direct tests.

### B. Daemon IPC Boundary Safety

- Status: Ready
- Evidence anchors:
  - daemon/src/ipc/protocol.rs
  - daemon/src/ipc/tests/ipc_framing.rs
  - daemon/src/ipc/tests/edge_cases.rs
  - daemon/src/ipc/tests/real_world.rs
- Why ready:
  - Framing validates malformed JSON, truncated payloads, and OOM-size payload rejection.
  - Roundtrip protocol tests include recon stateless and lab policy command/report paths.

### C. CLI Operator Surface

- Status: Ready
- Evidence anchors:
  - cli/src/main.rs
- Why ready:
  - Human and JSON render paths for policy presets are validated by tests.
  - CLI pathways are integrated with the daemon command model.

### D. CIDR and Self-Awareness Safety

- Status: Ready
- Evidence anchors:
  - utils/src/net/cidr.rs
  - utils/src/net/self_awareness.rs
  - utils/tests/self_awareness_tests.rs
- Why ready:
  - Non-contiguous netmask rejection and strict parsing behavior are enforced by tests.
  - Deterministic allow-list generation is present and regression-covered.

### E. Script Operations and CI Gate Stability

- Status: Ready
- Evidence anchors:
  - .github/workflows/scripts-quality.yml
  - scripts/tests/smoke_daemon_wrappers.sh
  - scripts/daemon/start_core_daemon.sh
  - scripts/systemd/install_service.sh
- Why ready:
  - Actionable ShellCheck gates are clean under CI configuration.
  - Wrapper smoke harness passes for daemon and systemd helper paths.

### F. Recon Telemetry and Fixture Baseline

- Status: Ready
- Evidence anchors:
  - recon/src/telemetry.rs
  - recon/tests/host_profile_fixture_tests.rs
  - recon/tests/fixtures/lab_host_profiles.json
- Why ready:
  - Stateless report formatting and canonical counters are directly tested.
  - Fixture invariants preserve expected lab host posture.

### Track 1 Exit Gates

Ship readiness for Track 1 is accepted only when all gates pass:

1. ./test.sh --all
2. shellcheck -x -e SC1091 -f gcc $(find scripts -type f -name '*.sh' | sort)
3. bash scripts/tests/smoke_daemon_wrappers.sh

## Track 2: Near-Ready With Bounded Effort

Track 2 is focused on reducing operator friction and increasing repeatability under drift-prone home and SMB environments.

### A. Preflight Runtime Checks

- Status: Ready at wrapper layer
- Target outcome:
  - Startup checks surface privilege, capability, and toolchain requirements before runtime failures.
- Anchors:
  - scripts/daemon/preflight.sh
  - scripts/daemon/start_core_daemon.sh
  - scripts/daemon/cli.sh
  - scripts/daemon/policy_preset_smoke.sh
  - docs/LOCAL_EXECUTION_GUIDE.md

### B. Policy Dry-Run and Explainability

- Status: Ready at policy-preset preview layer
- Target outcome:
  - Policy-impacting operations have explicit preview output and stable operator messaging.
- Anchors:
  - daemon/src/ipc/protocol.rs
  - cli/src/main.rs
  - scripts/daemon/policy_preset_smoke.sh
  - docs/LAB_HOST_POLICY_MATRIX.md

### C. Drift Reconciliation Workflow

- Status: Ready with fixture integrity checks
- Target outcome:
  - MAC-anchored identity and documented refresh loops mitigate DHCP drift breakage.
- Anchors:
  - recon/tests/fixtures/lab_host_profiles.json
  - recon/tests/host_profile_fixture_tests.rs
  - scripts/recon/drift_reconciliation_check.sh
  - docs/LAB_HOST_POLICY_MATRIX.md

### D. Operator Output Profiles

- Status: Ready for policy-preset command
- Target outcome:
  - Concise and verbose output modes for daily operations and diagnostics.
- Anchors:
  - cli/src/main.rs
  - scripts/daemon/policy_preset_smoke.sh
  - docs/LOCAL_EXECUTION_GUIDE.md

### Track 2 Exit Gates

1. All Track 1 gates pass.
2. First-run setup diagnosis can be completed in one pass from CLI or daemon output.
3. Drift reconciliation is documented and reproducible with fixture refresh steps.
4. Policy preview path exists for operator-impacting changes.

## Track 3: Roadmap and Environment-Conditional

Track 3 contains advanced capabilities that are valuable but not required for Track 1 or Track 2 release acceptance.

### A. Experimental Fuzz Module Completion

- Status: Ready with non-production compile constraints
- Current condition:
  - Deterministic fuzz primitives are implemented with unit tests.
  - Experimental fuzz feature is blocked for non-debug production builds.
- Anchors:
  - utils/src/fuzz/stateful_wifi_sim.rs
  - utils/src/fuzz/pcap_fuzzer.rs
  - utils/src/fuzz/packet_mutator.rs
  - utils/src/lib.rs
  - utils/Cargo.toml

### B. Hardware-Conditional Live Workflows

- Status: Ready with explicit degradation playbooks
- Current condition:
  - Live workflows remain hardware-dependent, but fallback behavior is now codified for constrained environments.
  - Operator guidance defines deterministic fallback validation when adapter capability or privilege is unavailable.
- Anchors:
  - utils/tests/live_wifi_tests.rs
  - utils/tests/live_cidr_self_awareness_smoke.rs
  - docs/LOCAL_EXECUTION_GUIDE.md
  - docs/LAB_HOST_POLICY_MATRIX.md
  - docs/EBPF_COMPILATION_GUIDE.md

### C. Public Beta and Packaging Expansion

- Status: Ready
- Current condition:
  - Baseline trust-doc artifacts are now published for threat model and privacy boundaries.
  - Reproducible packaging checklist is now documented and ready for iterative hardening.
  - Signed release-evidence bundles and distro transcripts are now published.
  - Reusable release-evidence script now generates signed manifests repeatably.
  - Privileged install/uninstall transcript is captured with passing daemon reachability checks.
- Anchors:
  - docs/SAVVY_OPERATOR_RELEASE_HANDOFF_PLAN.md
  - docs/SECURITY_MODEL.md
  - docs/PRIVACY_AND_TELEMETRY.md
  - docs/RELEASE_PACKAGING_CHECKLIST.md
  - docs/release_evidence/2026-06-09-rc0/SHA256SUMS
  - docs/release_evidence/2026-06-09-rc0/SHA256SUMS.asc
  - docs/release_evidence/2026-06-09-rc0/distro-validation-ubuntu-24.04.md
  - scripts/release/generate_release_evidence.sh
  - docs/release_evidence/2026-06-09-rc1/SHA256SUMS
  - docs/release_evidence/2026-06-09-rc1/SHA256SUMS.asc
  - docs/release_evidence/2026-06-09-rc1/distro-validation-ubuntu-24.04.md
  - docs/release_evidence/2026-06-09-rc1/distro-validation-ubuntu-24.04-privileged.md
  - docs/release_evidence/2026-06-09-rc2/SHA256SUMS
  - docs/release_evidence/2026-06-09-rc2/SHA256SUMS.asc
  - docs/release_evidence/2026-06-09-rc2/distro-validation-ubuntu-24.04-privileged.md
  - docs/ENTERPRISE_INTEGRATION.md
  - docs/DISTRO_GUIDES.md

### Track 3 Exit Gates

1. Track 1 and Track 2 exit gates pass and remain stable.
2. Experimental modules have deterministic tests or are explicitly constrained to non-production builds.
3. Hardware-conditional paths degrade gracefully with actionable operator messaging.

## Exclusions For Immediate Release Readiness

The following are intentionally excluded from immediate release readiness decisions:

- Experimental fuzz scaffolds under feature-gated paths in utils/src/fuzz.
- Hardware-only live tests that require external adapters or elevated runtime setup not present in baseline CI.
- Future roadmap extensions tracked in docs/SAVVY_OPERATOR_RELEASE_HANDOFF_PLAN.md.

## Finish Criteria: Readiness Complete

Readiness is considered complete when all criteria below are true:

1. Track 1 gates are consistently green locally and in CI.
2. Track 2 exit gates are documented as satisfied with reproducible operator steps.
3. Track 3 items are either completed or explicitly marked as deferred with non-production constraints.
4. The three companion docs remain aligned:
   - docs/BASTION_PRODUCTION_READINESS_MAP.md
   - docs/SAVVY_OPERATOR_RELEASE_HANDOFF_PLAN.md
   - docs/LOCAL_EXECUTION_GUIDE.md

## Immediate Next Milestones

1. Maintain recurring release evidence refresh cadence for each RC tag.
2. Expand privileged distro transcript coverage beyond Ubuntu baseline.
3. Continue Track 3 roadmap items that are explicitly environment-conditional or deferred by design.
