# Bastion Savvy-Operator Release Handoff Plan

This document is a post-hoc productization plan for shipping Bastion to:

- Savvy home network users (curious, technical, hands-on)
- Independent organizations and small-to-medium non-corporate networks

It is designed for direct handoff to either an AI implementation agent or a development team.

Companion operator runbook: `docs/LAB_HOST_POLICY_MATRIX.md`.

## Current Implementation Status (as of 2026-06-09)

1. M1 baseline safety goals are implemented and validated through preflight checks, policy preview paths, and drift reconciliation workflow docs.
2. M2 operator UX hardening goals are implemented for concise/verbose output profiles and deterministic day-2 playbooks.
3. M3 trust-doc and packaging artifacts are complete, including signed checksum evidence sets and a reusable release-evidence generator script.
4. Privileged install/uninstall rollback transcript has been captured with passing daemon reachability and policy smoke checks.
5. Deferred by developer decision: prioritize a future session focused only on strike/capture operator ergonomics and workflow polish.
6. RC2 confirmation cycle is complete with refreshed release evidence and passing privileged transcript validation.

## 1) Product Positioning

### Primary Positioning

Bastion is an operator-first network defense and visibility engine for users who want control, transparency, and technical depth.

### Explicit Non-Target

Do not position Bastion as a mass-market one-click consumer app for non-technical end users.

### Messaging Pillars

1. Defensive-first architecture
2. Transparent and inspectable behavior
3. Real network validation, not synthetic-only claims
4. Safe defaults with advanced controls available

## 2) Release Objectives

1. Preserve Bastion's technical power while reducing operational friction.
2. Make daily operator workflows reliable under real LAN conditions (DHCP churn, interface changes, privilege boundaries).
3. Ensure public-release trust via safety guardrails and clear documentation of failure modes.
4. Treat CIDR/subnet values as environment-specific and auto-detected by default (hardcoded ranges are examples only).

## 3) Guardrails (Must Not Regress)

1. Defensive workflow remains default and clearly separated from any offensive-adjacent modules.
2. No silent destructive actions without explicit operator opt-in.
3. Every security-impacting action has a dry-run or preview path where feasible.
4. Baseline and policy outputs remain deterministic and machine-checkable.

## 4) Workstreams

## A. Operator Experience

Deliverables:

1. Safe default mode with explicit advanced mode toggle.
2. Dry-run policy application with clear diff output.
3. Structured logs with concise and verbose profiles.
4. Reliable export/import of baseline artifacts.

Acceptance criteria:

1. Operator can run first-time baseline without guessing flags.
2. Operator can preview policy impact before apply.
3. Output is stable enough for CI assertions and scripted automation.

## B. Privilege and Runtime Ergonomics

Deliverables:

1. Root-required paths documented and validated at startup.
2. Clear runtime checks for missing capabilities (pcap, interface perms, toolchain path).
3. Actionable remediation messages (what failed, why, exact next command).

Acceptance criteria:

1. Privilege failures do not appear as silent or ambiguous scan failures.
2. Setup issues are diagnosable in one pass from CLI output.

## C. Drift Resilience (Home/SMB Reality)

Deliverables:

1. MAC-anchored identity handling in baseline guidance.
2. DHCP reservation recommendation embedded in policy workflow docs.
3. Baseline refresh command path for host IP drift.

Acceptance criteria:

1. Host IP churn does not force brittle failing tests.
2. Operators can reconcile drift with a documented 3-step process.

## D. Trust and Public Release Readiness

Deliverables:

1. Security model document (threat model, assumptions, non-goals).
2. Defensive posture statement in root README and CLI help.
3. Minimal telemetry/privacy statement (what Bastion stores, where, how long).

Acceptance criteria:

1. A technical reviewer can understand safety boundaries in under 10 minutes.
2. Public docs do not over-claim detection or prevention guarantees.

## E. Distribution and Packaging

Deliverables:

1. Reproducible release build instructions.
2. Installation pathways for common Linux environments.
3. Smoke-test script validating daemon + CLI + recon baseline path.

Acceptance criteria:

1. New operator can install and run first baseline in less than 30 minutes.
2. Release artifacts are versioned and verifiable.

## 5) Milestone Tracker (Strict)

The milestones below are sequenced and dependency-ordered. Do not start a milestone until all required dependencies are complete.

### Effort Scale

- S: <= 2 engineering days
- M: 3-5 engineering days
- L: 6-10 engineering days

### Owners

- Runtime Lead: daemon/runtime/privilege checks
- CLI Lead: command behavior, output profiles, dry-run UX
- QA Lead: regression tests, smoke scripts, acceptance checks
- Docs Lead: README/help/playbooks/release guidance
- Release Lead: packaging, versioning, beta process

### Master Milestone Table

| Milestone | Goal | Est. Effort | Depends On | Primary Owner | Status |
| --- | --- | --- | --- | --- | --- |
| M1 | Stability and safety baseline | L | None | Runtime Lead | Completed |
| M2 | Operator UX hardening | M | M1 | CLI Lead | Completed |
| M3 | Public beta readiness | M | M1, M2 | Release Lead | Completed |

## 6) Milestone Details

## M1: Stability and Safety Baseline

Target duration: 1-2 weeks

Scope:

1. Startup preflight checks for privilege/capability/dependency validation.
2. Actionable runtime remediation messages for pcap/interface/toolchain failures.
3. Explicit dry-run preview path for policy-impacting operations.
4. Drift resilience baseline using MAC-anchored identity guidance.
5. DHCP churn reconciliation documentation and fixture alignment.

Deliverables:

1. Preflight module integrated into daemon/CLI startup path.
2. Standardized error text format: failure, cause, remediation command.
3. Dry-run output contract documented and test-covered.
4. "DHCP churn and host identity reconciliation" playbook in docs.

Acceptance checks:

1. Privilege failures never surface as silent scan empties.
2. Operators can complete setup diagnosis in one execution pass.
3. Policy-impacting paths provide preview output before apply.
4. Fixture/test baselines tolerate host IP drift through documented refresh flow.

Suggested verification commands:

1. `./test.sh --daemon`
2. `./test.sh --cli`
3. `./test.sh --recon`

Exit gate:

No known unsafe defaults and no critical setup ambiguity remains.

## M2: Operator UX Hardening

Target duration: 1 week

Depends on: M1

Scope:

1. Concise vs verbose output profiles for core workflows.
2. Baseline export/import polish with deterministic structure.
3. Failure-mode playbooks for common home/SMB breakpoints.

Deliverables:

1. Stable CLI output profiles (`human`, `json`, verbose diagnostics).
2. Export/import path with validation and failure hints.
3. Operator runbooks for router reset, DHCP churn, privilege drift.

Acceptance checks:

1. First-time baseline can be completed without source inspection.
2. Day-2 operator workflows are scriptable and deterministic.
3. Failure-mode docs map directly to observed runtime errors.

Suggested verification commands:

1. `./test.sh --cli`
2. `./test.sh --recon`
3. `./test.sh --all`

Exit gate:

Repeatable day-2 operations by savvy operators without ad-hoc tribal knowledge.

## M3: Public Beta Readiness

Target duration: 1 week

Depends on: M1 and M2

Scope:

1. Reproducible packaging and release artifact verification.
2. Public trust docs: threat model, defensive posture, telemetry/privacy.
3. Beta feedback loop and issue triage rubric.

Deliverables:

1. Release checklist and signed/tagged artifact process.
2. Security model and non-goals document linked from README.
3. Public beta operator onboarding guide.

Acceptance checks:

1. New savvy operator can install and baseline in < 30 minutes.
2. Release artifacts are versioned, reproducible, and reversible.
3. Public docs avoid over-claims and clearly state boundaries.

Suggested verification commands:

1. `./test.sh --all`
2. `scripts/daemon/policy_preset_smoke.sh --format human`
3. `scripts/daemon/policy_preset_smoke.sh --format json`

Exit gate:

Public beta can run safely in home/SMB labs with predictable outcomes.

### M3 Kickoff Artifacts (Implemented)

1. `docs/SECURITY_MODEL.md` (threat model, assumptions, non-goals).
2. `docs/PRIVACY_AND_TELEMETRY.md` (data handling and retention policy).
3. `docs/RELEASE_PACKAGING_CHECKLIST.md` (reproducible build, verification, rollback).

## 7) Dependency Order (Task Graph)

1. M1.T1 Preflight checks -> required for M1.T2 and M2.T3
2. M1.T2 Runtime remediation messages -> required for M2.T3
3. M1.T3 Dry-run policy preview -> required for M2.T1
4. M1.T4 Drift reconciliation docs/fixtures -> required for M2.T2
5. M2.T1 Output profiles -> required for M3 onboarding docs
6. M2.T2 Export/import polish -> required for M3 release checklist
7. M2.T3 Failure-mode playbooks -> required for M3 trust docs

## 8) Definition of Done (Public Beta)

Bastion is public-beta ready only when all conditions are met:

1. M1, M2, and M3 exit gates are all satisfied.
2. `./test.sh --all` passes on release candidate.
3. Smoke scripts pass on at least one representative home/SMB Linux environment.
4. README and CLI help clearly state defensive-first intent and operational boundaries.
5. Rollback steps are documented and tested at least once.

## 9) AI/Team Execution Protocol

Required protocol for each milestone:

1. Implement in small PR-sized increments.
2. Run milestone-scoped tests after each increment.
3. Add/update docs in the same change set as behavior changes.
4. Include a short risk note: regression risk, operational risk, rollback note.
5. Do not begin next milestone until current milestone exit gate is explicitly recorded as met.
