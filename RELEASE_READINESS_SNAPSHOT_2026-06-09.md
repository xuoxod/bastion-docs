# Bastion Release Readiness Snapshot (2026-06-09)

## Overall Status

- Release readiness checkpoint: PASS
- Scope: RC2 packaging, signatures, and privileged distro validation confirmation
- Branch baseline: `main`

## Gate Summary

1. Workspace tests: pass
2. Script lint gate (ShellCheck): pass on latest scripts commit
3. Wrapper smoke harness: pass
4. Privileged systemd transcript: pass (install, start, socket ready, CLI ping/status, policy smoke human/json, stop, uninstall)

## Evidence Bundle (RC2)

- `docs/release_evidence/2026-06-09-rc2/release-artifacts.txt`
- `docs/release_evidence/2026-06-09-rc2/toolchain-metadata.txt`
- `docs/release_evidence/2026-06-09-rc2/SHA256SUMS`
- `docs/release_evidence/2026-06-09-rc2/SHA256SUMS.asc`
- `docs/release_evidence/2026-06-09-rc2/signature-verify.txt`
- `docs/release_evidence/2026-06-09-rc2/RELEASE_SIGNING_PUBLIC_KEY.asc`
- `docs/release_evidence/2026-06-09-rc2/distro-validation-ubuntu-24.04-privileged.md`
- `docs/release_evidence/2026-06-09-rc2/systemd-rollback-ubuntu-24.04.log`

## Prior Candidate Archive

- RC1 artifacts remain available under `docs/release_evidence/2026-06-09-rc1/`.

## Release-Critical Fixes Captured In This Checkpoint

1. Systemd service args aligned to daemon-supported flags.
2. Runtime interface auto-selection injected during install.
3. eBPF bytecode installed to absolute runtime path and passed via `--bytecode`.
4. Systemd hardening adjusted so daemon can create UDS socket at `/tmp/bastion_master.sock`.
5. Privileged transcript helper now includes socket readiness wait and automatic service diagnostics on failure.

## Operational Note

The privileged transcript intentionally uninstalls the service at the end to validate rollback. Seeing `Unit bastion.service could not be found` after transcript completion is expected.

## Next Recommended Actions

1. Repeat evidence generation for each new RC tag and append distro coverage.
2. Keep this snapshot updated whenever release acceptance criteria change.
