# Bastion Privacy and Telemetry Statement

This statement defines what Bastion stores during normal operator workflows.

## Data Collected

Bastion may store or emit:

1. Operator-requested policy and status outputs.
2. Local scan/baseline artifacts (for example CSV/JSON reports produced by commands or scripts).
3. Runtime logs from daemon and helper scripts when executed.

Bastion does not require a cloud backend for baseline local operation.

## Data Locations

Typical local locations include:

1. Workspace-generated report files under project or operator-selected output paths.
2. Local daemon socket and runtime files (for example under `/tmp` during active sessions).
3. System journal entries when running as a systemd-managed service.

## Retention Model

1. Bastion does not enforce long-term remote telemetry retention.
2. Retention of local files and logs is controlled by the operator and host policy.
3. Operators should periodically prune report artifacts and logs per environment policy.

## Telemetry Boundaries

1. No claim of mandatory remote analytics pipeline in baseline workflows.
2. Any export/share behavior is operator-initiated.
3. CI/test output is local to the execution environment unless the operator publishes it.

## Operator Hygiene Recommendations

1. Keep generated reports in controlled directories with least-privilege access.
2. Rotate or clear verbose logs after incident triage completion.
3. Avoid committing sensitive environment-specific host data to public repositories.

## Verification Commands

```bash
./test.sh --all
bash scripts/tests/smoke_daemon_wrappers.sh
```

These commands verify operational integrity, not legal/privacy compliance for every deployment jurisdiction.
