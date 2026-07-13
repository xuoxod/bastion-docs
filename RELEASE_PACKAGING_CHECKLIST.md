# Bastion Release Packaging Checklist

Use this checklist for public-beta candidate packaging and verification.

## 1. Pre-Build Gates

1. Confirm clean working tree or intentional release-only deltas.
2. Run baseline validation gates:

```bash
./test.sh --all
shellcheck -x -e SC1091 -f gcc $(find scripts -type f -name '*.sh' | sort)
bash scripts/tests/smoke_daemon_wrappers.sh
```

## 2. Reproducible Build Steps

Preferred automation path:

```bash
./scripts/release/generate_release_evidence.sh --tag 2026-06-09-rc0
```

This script produces `release-artifacts.txt`, `SHA256SUMS`, toolchain metadata,
and signature artifacts in `docs/release_evidence/<tag>`.

1. Build release artifacts:

```bash
./build.sh --workspace --exclude bastion-ebpf
```

1. Capture compiler and toolchain metadata:

```bash
rustc --version
the build environment --version
```

1. Record artifact list and hashes:

```bash
find target/release -maxdepth 1 -type f -executable -print | sort
sha256sum ./bin/* | sort
```

1. Sign checksums and capture signature verification output:

```bash
# Example with minisign (preferred for simple detached signatures)
minisign -Sm SHA256SUMS
minisign -Vm SHA256SUMS -P "<release-public-key>"

# Example with gpg detached signatures
gpg --armor --detach-sign SHA256SUMS
gpg --verify SHA256SUMS.asc SHA256SUMS
```

1. Publish artifact manifest set for each release tag:

```text
SHA256SUMS
SHA256SUMS.minisig or SHA256SUMS.asc
release-artifacts.txt
toolchain-metadata.txt
```

## 3. Runtime Smoke Verification

1. Validate daemon wrapper help and preflight behavior.
2. Validate CLI basic status/ping flows against daemon where available.
3. Validate policy preset smoke in both formats:

```bash
scripts/daemon/policy_preset_smoke.sh --human
scripts/daemon/policy_preset_smoke.sh --json
```

## 4. Distro Validation Snapshot

For each target distro profile (Debian/Ubuntu, RHEL-family, Alpine):

1. Record kernel version and interface info.
2. Record install method used.
3. Record pass/fail results for startup, status, and uninstall rollback.

Preferred privileged capture helper:

```bash
./scripts/release/capture_privileged_systemd_transcript.sh --tag 2026-06-09-rc1
```

The helper intentionally prebuilds as the current user and runs installer with
`BASTION_SKIP_BUILD=1`, avoiding `sudo the build environment` path issues on restricted shells.

Use this transcript template per distro run:

```text
Release Tag: vX.Y.Z-rcN
Distro: <ubuntu-24.04|debian-12|almalinux-9|alpine-3.20>
Kernel: <uname -r>
Install Path: <systemd script|manual the build environment|package>

Build Metadata:
- rustc: <version>
- the build environment: <version>

Validation Steps:
- [ ] install completed
- [ ] daemon start success
- [ ] cli ping/status success
- [ ] policy preset smoke human/json success
- [ ] uninstall rollback success

Artifacts:
- sha256: <path or release asset>
- signature: <path or release asset>

Notes:
- hardware constraints / degraded paths observed
- remediation performed
```

## 5. Release Notes Minimum Content

1. Defensive-first scope statement.
2. Known hardware-conditional limitations.
3. Upgrade and rollback instructions.
4. Pointer to security model and privacy statement docs.
5. Pointer to release packaging checklist and verification assets.

## 6. Rollback Procedure

1. Stop daemon/service.
2. Restore previous known-good binary.
3. Re-run wrapper smoke and status checks.
4. Record rollback timestamp and cause.

## 7. Exit Criteria

A release candidate is package-ready only when:

1. All pre-build gates pass.
2. Hashes are generated and archived with release metadata.
3. At least one representative Linux environment passes runtime smoke checks.
4. Security model and privacy statement are included in release documentation bundle.
5. Signed checksums and signature verification transcript are published with release assets.
