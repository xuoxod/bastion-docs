# Distro Validation Transcript

Release Tag: 2026-06-09-rc0
Distro: ubuntu-24.04
Kernel: 6.17.0-35-generic
Install Path: local cargo release build (non-systemd install path for this transcript)

## Build Metadata

- rustc: rustc 1.90.0 (1159e78c4 2025-09-14)
- cargo: cargo 1.90.0 (840b83a10 2025-07-30)
- artifact manifest: docs/release_evidence/2026-06-09-rc0/release-artifacts.txt
- checksums: docs/release_evidence/2026-06-09-rc0/SHA256SUMS
- detached signature: docs/release_evidence/2026-06-09-rc0/SHA256SUMS.asc
- public signing key: docs/release_evidence/2026-06-09-rc0/RELEASE_SIGNING_PUBLIC_KEY.asc
- verification transcript: docs/release_evidence/2026-06-09-rc0/signature-verify.txt

## Validation Steps

- [x] release build completed (`cargo build --release --workspace --exclude bastion-ebpf`)
- [x] checksum manifest generated
- [x] detached signature generated and verified
- [x] baseline software gate (`cargo test --workspace`) passed in same session
- [x] shell gate (`shellcheck` + wrapper smoke harness) passed in same session
- [ ] systemd install/uninstall rollback run (not executed in this transcript)
- [ ] root daemon live runtime validation (not executed in this transcript)

## Notes

- eBPF crate requires specialized target/runtime and was intentionally excluded from this release bundle build path.
- This transcript is sufficient for packaging pipeline evidence, but not a privileged deployment acceptance record.
