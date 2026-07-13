# Bastion RC2 Release Notes (2026-06-09)

## Summary

Bastion RC2 is release-ready and freezes the post-RC1 hardening cycle.

This candidate confirms:

- reproducible packaging evidence generation
- signed checksum verification artifacts
- privileged systemd runtime validation with full lifecycle pass

## Validation Highlights

- Workspace tests passed (`cargo test --workspace`)
- Script lint gate passed (`shellcheck -x -e SC1091 ...`)
- Wrapper smoke harness passed (`bash scripts/tests/smoke_daemon_wrappers.sh`)
- Privileged transcript passed end-to-end:
  - install completed
  - service started
  - daemon socket reached ready state
  - CLI ping and status succeeded
  - policy preset smoke checks succeeded (human/json)
  - stop and uninstall rollback succeeded

## Evidence Bundle

- docs/release_evidence/2026-06-09-rc2/release-artifacts.txt
- docs/release_evidence/2026-06-09-rc2/toolchain-metadata.txt
- docs/release_evidence/2026-06-09-rc2/SHA256SUMS
- docs/release_evidence/2026-06-09-rc2/SHA256SUMS.asc
- docs/release_evidence/2026-06-09-rc2/signature-verify.txt
- docs/release_evidence/2026-06-09-rc2/RELEASE_SIGNING_PUBLIC_KEY.asc
- docs/release_evidence/2026-06-09-rc2/distro-validation-ubuntu-24.04-privileged.md
- docs/release_evidence/2026-06-09-rc2/systemd-rollback-ubuntu-24.04.log

## Notes

This checkpoint focuses on operator reliability and release evidence integrity.

Advanced wireless strike/capture ergonomics remain a documented follow-up focus for a future session.
