# Bastion -> RMEDIAtech Compatibility Sign-Off

Date: 2026-06-09

## Scope

This sign-off covers Bastion discovery JSON output consumed by RMEDIAtech upload and scan parsing paths.

## Contract Baseline

- Envelope fields:
  - report_type: string
  - netscan_version: string
  - timestamp: string
  - data: array
- Device fields required for compatibility path:
  - IP
  - MAC (optional)
  - Hostname (optional)
  - Timestamp
  - os (optional string)

## Verified Controls

1. Producer compatibility serialization:
   - `utils/src/net/report.rs` serializes `os` as string when present.
2. Fixture-backed contract tests:
   - `utils/tests/rmediatech_contract_tests.rs`
   - `utils/tests/fixtures/rmediatech/discovery_envelope_valid.json`
   - `utils/tests/fixtures/rmediatech/discovery_envelope_no_os.json`
3. CI gate:
   - `.github/workflows/rmediatech-contract.yml` runs the contract tests on PR/push for relevant paths.
   - The workflow installs `libpcap-dev` because the `utils` test binary links against `libpcap`.

## Known Boundaries

1. RMEDIAtech upload handlers currently accept `.json` and `.jsonl`.
2. CSV uploads are currently rejected by RMEDIAtech handlers and are not included in this sign-off.
3. Local or CI environments running `./test.sh --contract` need `libpcap` development headers available for linking.

## Acceptance Status

- Discovery JSON contract: Signed off.
- CSV upload contract: Not in scope until RMEDIAtech adds CSV ingest support.
