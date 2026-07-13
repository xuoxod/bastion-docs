# Bastion Lab Host Policy Matrix

This matrix captures observed LAN host behavior from a lab sweep and translates it into practical Bastion policy actions.

## Scope

- Capture window: 2026-06-08
- Source interface: wlp0s20f3
- Lab snapshot CIDR (example environment): 192.168.1.0/24
- Ground-truth fixture: recon/tests/fixtures/lab_host_profiles.json

## Host Classes

| Host | Role | Keep Open | Monitor | Restrict / Alert |
| --- | --- | --- | --- | --- |
| 192.168.1.1 | Fios router | TCP 53,80,443 and UDP 53,123 | UDP 1900/5353 activity volume | Alert on unexpected new TCP listeners (e.g., 8080,8443) |
| 192.168.1.159 | HP ENVY 6400 printer | TCP 631,9100 and management 80/443/8080 | UDP 161 (SNMP), UDP 5353, print queue behavior | Alert on internet-facing exposure and non-print lateral flows |
| 192.168.1.160 | Linux laptop (custom SSH) | TCP 30778 | UDP discovery chatter (open|filtered set) | Alert on any additional open inbound TCP services |
| 192.168.1.155 | Espressif/Amazon IoT node | UDP 5353 only (observed) | Matter + meshcop announcements | Alert on new inbound TCP listeners or SNMP/UPnP activation |
| 192.168.1.165 | Spotify endpoint | TCP 4070, UDP 5353 | meshcop and Spotify control plane traffic | Alert on privileged admin ports appearing |
| 192.168.1.166 | Spotify endpoint | TCP 4070,8888 and UDP 5353 | UDP 123/137/161 open|filtered drift | Alert on expansion outside media-control profile |
| 192.168.1.167 | Spotify/Amazon endpoint | TCP 4070 and UDP 5353 | meshcop and Spotify advertisements | Alert on additional persistent TCP listeners |

## Bastion Enforcement Guidance

1. Build allow-lists per host class rather than global flat rules.
2. Treat open|filtered UDP as low-confidence exposure and re-sample before hard-blocking.
3. Keep printer services segmented; enforce policy to prevent non-print lateral movement.
4. Keep laptop profile narrow: single inbound SSH port plus optional discovery traffic.
5. For IoT/media nodes, baseline expected mDNS/Matter/Spotify patterns and alert on drift.
6. Reserve DHCP leases for policy-critical hosts (for example the custom SSH laptop) so fixture IPs do not churn after router resets.

## Regression Alignment

The fixture-backed tests in recon/tests/host_profile_fixture_tests.rs ensure this observed baseline remains stable and machine-checkable while policy code evolves.

## Operator Runbook Companion

Use this section for day-2 operations when network state changes after router resets, DHCP lease churn, or device replacement.

## Quick Validation Loop

1. Confirm baseline tests pass.
2. Confirm currently live hosts on the subnet.
3. Verify policy-critical ports on only live hosts.
4. If drift exists, refresh fixture/IP mapping and re-run tests.

Suggested commands:

```bash
# Baseline fixture assertions
./test.sh --recon

# Scripted drift reconciliation check
./scripts/recon/drift_reconciliation_check.sh

# Detect active interface CIDR dynamically
CIDR=$(ip -o -4 addr show scope global | awk '{print $4}' | head -n1)

# Live hosts only
nmap -sn "$CIDR"

# Example targeted validation for policy-critical laptop port
nmap -Pn -sT -p 30778 192.168.1.160
```

## DHCP Drift Reconciliation

When a known host moves from one IP to another:

1. Confirm identity by MAC and hostname, not IP only.
2. Update fixture host IP to match current lease.
3. Keep role label unchanged unless the device itself changed.
4. Re-run fixture and workspace tests.
5. Add or verify router DHCP reservation for policy-critical hosts.

Identity check guidance:

1. Match at least two of: MAC, hostname, service signature.
2. Treat IP-only matches as untrusted during churn windows.
3. If identity is uncertain, do not auto-apply restrictive policy updates.

## Incident Triage Rules

Apply the following response rules for observed drift:

| Condition | Response | Escalation Threshold |
| --- | --- | --- |
| Known host, same MAC, new IP | Update fixture and reservation | Escalate if repeats more than 2 times in 7 days |
| Known host, same IP, new open TCP listener | Mark as suspicious and alert | Escalate immediately for privileged/admin ports |
| Unknown MAC appears with open services | Quarantine policy candidate | Escalate if persists across 2 scans |
| Printer/IoT node gains broad TCP exposure | Restrict and alert | Escalate immediately |

## Change Management Protocol

For every baseline update:

1. Record timestamp, reason, and affected host label.
2. Keep changes scoped to observed drift; avoid unrelated fixture edits.
3. Run and record the exact validation command set.
4. Commit with a drift-specific message (for example: `recon: update lab fixture after DHCP drift`).

## Fixture Integrity Acceptance Checks

The fixture baseline is accepted only when all checks pass:

1. Host labels are unique.
2. Host IPs are unique for the snapshot window.
3. Identity fields (`label`, `ip`, `mac_vendor`) are non-empty.
4. Top-level `network` and `source_interface` fields are populated.

These checks are enforced in:

- `recon/tests/host_profile_fixture_tests.rs`

## Routine Maintenance Cadence

1. Weekly: run host and policy-critical port validation.
2. After any router reset: run full drift reconciliation workflow.
3. Before release milestones: run full workspace tests and policy smoke script.

Suggested pre-release command set:

```bash
./test.sh --cli
./test.sh --daemon
./test.sh --recon
./scripts/recon/drift_reconciliation_check.sh
./test.sh --all
scripts/daemon/policy_preset_smoke.sh --format human
scripts/daemon/policy_preset_smoke.sh --format json
```
