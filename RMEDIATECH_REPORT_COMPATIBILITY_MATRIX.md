# Bastion <-> RMEDIAtech Report Compatibility Matrix

## Objective

Define the exact, code-verified compatibility contract between Bastion report outputs and RMEDIAtech upload/display parsers.

## Ground Truth Sources

- Bastion producer logic:
  - utils/src/net/report.rs
  - utils/src/bin/discover.rs
- RMEDIAtech consumer logic:
  - internal/scanlogic/scanlogic.go
  - internal/handlers/uploader.go
  - internal/handlers/action_studio_upload.go
  - views/user/report.jet

## Key Findings

1. RMEDIAtech upload handlers currently accept only `.json` and `.jsonl`.
2. CSV reports are not currently accepted by RMEDIAtech upload endpoints.
3. RMEDIAtech discovery parser expects unified envelope shape:
   - report_type
   - netscan_version
   - timestamp
   - data (array of host objects)
4. Host object fields expected by RMEDIAtech discovery path include:
   - IP, IPv6, MAC, Hostname, Timestamp
   - Optional booleans and metadata fields
   - `os` must deserialize as a string (not a nested object)

## Compatibility Matrix

| Field | Bastion (after patch) | RMEDIAtech expects | Status |
|---|---|---|---|
| report_type | discovery | discovery | Compatible |
| netscan_version | string | string | Compatible |
| timestamp | RFC3339 string | string | Compatible |
| data | array | array | Compatible |
| Host IP | `IP` | `IP` | Compatible |
| Host MAC | `MAC` | `MAC` | Compatible |
| Host name | `Hostname` | `Hostname` | Compatible |
| Host OS | `os` string | `os` string | Compatible |

## Important Scope Clarification

- RMEDIAtech report uploads currently reject CSV by extension.
- Bastion CSV output remains useful for operator exports and tooling, but not for direct RMEDIAtech upload unless RMEDIAtech adds CSV parsing support.
- Contract test environments must provide `libpcap` development libraries because the `utils` test target links against `libpcap`.

## Recommended Next Steps

1. Keep discovery JSON as canonical RMEDIAtech integration format.
2. Enforce CI checks in Bastion for RMEDIAtech contract-sensitive fields (`report_type`, envelope shape, `os` string) via `.github/workflows/rmediatech-contract.yml`.
3. Maintain fixture-backed contract tests in `utils/tests/rmediatech_contract_tests.rs` with sample payloads in `utils/tests/fixtures/rmediatech/`.
4. If CSV upload is desired in RMEDIAtech, implement explicit CSV ingest path there first, then add Bastion CSV schema parity tests.
