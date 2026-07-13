# Bastion Security Model

This document defines Bastion security boundaries for public-beta deployments.

## Scope

Bastion is a defensive-first network control plane with:

1. Daemon runtime for privileged data-plane actions.
2. CLI and scripts as operator control surface.
3. Optional feature-gated experimental modules that are not production defaults.

## Threat Model

### In Scope Threats

1. Untrusted network traffic reaching host interfaces.
2. Malformed CLI/IPC payloads that attempt daemon crashes or parser abuse.
3. Host drift conditions that can cause policy misapplication (IP churn, interface changes).
4. Operator misconfiguration in privilege-sensitive workflows.

### Out of Scope Threats

1. Physical host compromise.
2. Kernel or firmware zero-day exploitation.
3. Compromised root account or local admin token theft.
4. Security guarantees for offensive or experimental workflows outside defensive defaults.

## Assumptions

1. Operator controls the host and its package sources.
2. Host OS applies normal patching and hardening practices.
3. Root-only pathways are executed intentionally by trusted operators.
4. Baseline and policy docs are followed for drift reconciliation.

## Security Boundaries

1. Defensive workflow is default; no implicit offensive behavior.
2. Policy-affecting actions expose preview/dry-run or explicit intent messaging.
3. IPC framing rejects malformed and oversized payloads before state mutation.
4. Experimental fuzz behavior is feature-gated and blocked for non-debug production builds.

## Failure and Degradation Model

1. Missing privileges should fail fast with actionable guidance.
2. Missing hardware capability should degrade to non-hardware validation paths.
3. Missing daemon socket should return explicit connection remediation, never silent success.
4. Live hardware tests may be skipped in constrained CI/runtime environments.

## Non-Goals

1. Bastion is not marketed as autonomous, zero-config consumer security software.
2. Bastion does not claim guaranteed prevention for all attack classes.
3. Bastion does not replace host patch management or endpoint hardening controls.

## Operator Verification Commands

```bash
./test.sh --all
shellcheck -x -e SC1091 -f gcc $(find scripts -type f -name '*.sh' | sort)
bash scripts/tests/smoke_daemon_wrappers.sh
```

Passing these commands validates baseline guardrails but does not imply full host hardening.
