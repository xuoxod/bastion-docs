# Bastion Tactical Network Scanner: Comprehensive Usage & Operations Guide

This guide provides an exhaustive operations manual for Bastion's TCP and UDP scanning utilities. It details all supported modes, signature matching behaviors, command-line arguments, constraints, and all possible scanner CLI option combinations.

---

## 📖 Table of Contents
1. [Architecture Overview](#-architecture-overview)
2. [TCP Stealth & Service Scanner (`scan`)](#-tcp-stealth--service-scanner-scan)
   - [CLI Commands & Options](#cli-commands--options)
   - [Underlying Mechanics (SYN vs Connect)](#underlying-mechanics-syn-vs-connect)
   - [OS-Targeted Port Profiles](#os-targeted-port-profiles)
   - [Exhaustive TCP Command Permutation Matrix](#exhaustive-tcp-command-permutation-matrix)
3. [UDP Tactical & Active Scout Scanner (`udp`)](#-udp-tactical--active-scout-scanner-udp)
   - [CLI Commands & Options](#cli-commands--options-1)
   - [Active Protocol Probes Database](#active-protocol-probes-database)
   - [Exhaustive UDP Command Permutation Matrix](#exhaustive-udp-command-permutation-matrix)
4. [Layer 2 Discovery Engine (`discover`)](#-layer-2-discovery-engine-discover)
   - [CLI Commands & Options](#cli-commands--options-2)
   - [ARP/NDP Mechanics & OS Sniffing](#arpndp-mechanics--os-sniffing)
   - [Exhaustive Discovery Execution Examples](#exhaustive-discovery-execution-examples)
5. [Standalone Binary Compilation & Distribution](#-standalone-binary-compilation--distribution)

---

## 🏗 Architecture Overview

Bastion's scanning suite is built on decoupled, high-performance crates following **Separation of Concerns (SOC)** and the **One Job Rule (OJR)**:

```mermaid
graph TD
    CLI_TCP[scan CLI] -->|runs| TcpScanner[utils::net::TcpScanner]
    CLI_UDP[udp CLI] -->|runs| UdpScanner[utils::net::UdpScanner]
    CLI_UDP -->|runs| UdpActive[utils::net::UdpActiveScanner]
    
    TcpScanner -->|packet forge| Forge[forge Crate]
    TcpScanner -->|packet inject| Strike[strike Crate]
    
    TcpScanner -->|banner/active grab| ReconSvc[recon::services::active_fingerprint]
    ReconSvc -->|identifies| FingerprintCrate[fingerprint Crate (SST)]
    
    UdpActive -->|matches payload| FingerprintCrate
```

* **`fingerprint` Crate**: The absolute **Single Source of Truth (SST)** for OS fingerprinting. It contains signature parsing, heuristic window-size matching, and estimates original packet TTLs while filtering out control traffic false positives.
* **`forge` & `strike` Crates**: Handle raw packet construction (SYN/ACK layers) and raw socket injection.
* **`utils` Crate**: Hosts the execution engines (`TcpScanner`, `UdpScanner`, `UdpActiveScanner`) and CLI binaries.

---

## 🔒 TCP Stealth & Service Scanner (`scan`)

The `scan` binary is a high-speed TCP port scanner capable of half-open stealth scans and deep banner-grabbing service identification.

### CLI Commands & Options

```
Usage: scan <target_ip> [ports] [options]
```

| Argument / Option | Short | Long | Type | Description |
| :--- | :--- | :--- | :--- | :--- |
| `<target_ip>` | - | - | Positional | The target IPv4 or IPv6 address to scan. |
| `[ports]` | - | - | Positional | Comma-separated list/ranges of ports (e.g., `22,80,443-445,8080`). |
| Merged Defaults | `-d` | `--with-defaults` | Flag | Merges custom ports with default surface ports (`1-1024`). |
| Smart Profile | `-o` | `--os` | String | Auto-injects common ports for specific target OS (`linux`, `windows`, `android`, `ios`, `network`). |
| Stealth Only | `-s` | `--stealth-only` | Flag | Performs a 100% silent scan (skips active banner grabbing / connection opens). |
| Scan Method | `-m` | `--method` | String | Force scanning mode (`auto`, `syn`, `connect`). Defaults to `auto`. |
| Export File | `-w` | `--write` | String | Path to write the tactical CSV report (creates parent folders automatically). |

---

### Exhaustive TCP Command Permutation Matrix

Every port input source (`ports`, `--with-defaults`, and `--os`) is cumulative; the scanner merges all resolved target ports, sorts them, and deduplicates the list before scanning. Below is the exhaustive matrix of all possible command-line combinations for the TCP scanner:

#### 1. Basic Single Target & Custom Ports Scan
Scan only a specified target and port list with default automated privilege check and active banner grabbing:
```bash
./target/release/scan 192.168.1.1 22,80,443
```

#### 2. Target & Custom Port Range Scan
Scan a targeted target and sequential port ranges:
```bash
./target/release/scan 192.168.1.1 8000-8080
```

#### 3. Target & Default Ports Scan
Scan only the system's default well-known ports (1–1024):
```bash
./target/release/scan 192.168.1.1 --with-defaults
```

#### 4. Target, Custom Ports, & Default Ports Merged
Scan custom ports merged with the default well-known ports:
```bash
./target/release/scan 192.168.1.1 8080,9000 --with-defaults
```

#### 5. Target & Smart OS Port Profile Scan
Scan ports optimized for a specific target operating system (e.g., Android):
```bash
./target/release/scan 192.168.1.185 --os android
```

#### 6. Target, Smart OS Profile, & Custom Ports Merged
Scan OS-specific ports merged with custom user-provided ports:
```bash
./target/release/scan 192.168.1.185 30778 --os android
```

#### 7. Target, Smart OS Profile, Custom Ports, & Default Ports Merged
Merge all port inputs (defaults 1–1024, smart OS profile, and custom ports) into a single unified scan list:
```bash
./target/release/scan 192.168.1.185 30778,50000 --os android --with-defaults
```

#### 8. Pure Stealth Scan (Skip Banners)
Skip the active banner grabbing phase entirely to execute a 100% quiet scan (SYN or Connect only):
```bash
./target/release/scan 192.168.1.160 22,80,443 --stealth-only
```
*Note: `--stealth-only` can be combined with **any** port option configuration (examples 1 through 7).*

#### 9. Forced TCP Connect Scan (No root privileges required)
Forces standard user-space TCP Connect scanning:
```bash
./target/release/scan 192.168.1.1 22,80,443 --method connect
```
*Note: `--method connect` can be combined with **any** port option configuration (examples 1 through 7) and `--stealth-only`.*

#### 10. Forced Raw SYN Scan (Requires root privileges)
Forces the raw SYN packet scan (will fail if not executed with `sudo`):
```bash
sudo ./target/release/scan 192.168.1.1 22,80,443 --method syn
```
*Note: `--method syn` can be combined with **any** port option configuration (examples 1 through 7) and `--stealth-only`.*

#### 11. Custom Port Scan with CSV Export
Writes the scan output directly to a CSV report file:
```bash
./target/release/scan 192.168.1.1 22,80,443 --write reports/scan_report.csv
```
*Note: `--write` can be combined with **any** scanning combination (examples 1 through 10).*

---

## ⚡ UDP Tactical & Active Scout Scanner (`udp`)

UDP scanning is connectionless. The `udp` utility sweeps ports and actively sends specialized protocol payloads to identify services.

### CLI Commands & Options

```
Usage: udp <target_ip> [ports] [options]
```

| Argument / Option | Short | Long | Type | Description |
| :--- | :--- | :--- | :--- | :--- |
| `<target_ip>` | - | - | Positional | The target IPv4 or IPv6 address to scan. |
| `[ports]` | - | - | Positional | Comma-separated list/ranges of ports (e.g., `53,123,161,500`). |
| Merged Defaults | `-d` | `--with-defaults` | Flag | Merges custom ports with default surface ports (`1-1024`). |
| Custom Payload | `-p` | `--payload` | String | String payload to transmit as a custom probe (disabled in active scout mode). |
| Timeout | `-t` | `--timeout-ms` | u64 | Time to wait for a response in milliseconds. (Default: `800ms`). |
| Concurrency Limit | `-l` | `--limit` | usize | Maximum simultaneous asynchronous UDP probes. (Default: `1000`). |
| Active Scouting | `-s` | `--scout` | Flag | Enables sending all active protocol signatures to the specified ports. |
| Specific Protocols | `-P` | `--protocols` | String | Limit scouting to specific comma-separated protocols (`dns`, `ntp`, `snmp`, `ssdp`, `echo`, `coap`, `tftp`, `sip`, `ike`). |
| Export File | `-w` | `--write` | String | Path to write the tactical CSV report. |

---

### Exhaustive UDP Command Permutation Matrix

UDP scanning supports two modes: **Standard Sweep** (sends raw or custom payloads) and **Active Protocol Scouting** (sends exact protocol-specific queries). 

* **CRITICAL CONSTRAINT**: You **cannot** combine a custom payload (`-p`/`--payload`) with active scouting (`-s`/`--scout` or `-P`/`--protocols`). Doing so will result in an immediate CLI error exit.

Below is the exhaustive matrix of all possible command-line combinations for the UDP scanner:

#### 1. Standard UDP Scan (Specific Custom Ports)
Scan only custom ports using a default empty payload and standard timeouts:
```bash
./target/release/udp 192.168.1.1 53,123,161
```

#### 2. Standard UDP Scan (Default Ports Only)
Scan only default surface ports (1–1024) with standard parameters:
```bash
./target/release/udp 192.168.1.1 --with-defaults
```

#### 3. Standard UDP Scan (Custom & Default Ports Merged)
Merge custom ports and well-known defaults:
```bash
./target/release/udp 192.168.1.1 5060,16161 --with-defaults
```

#### 4. Custom Payload UDP Scan
Scan specific ports and send a custom string payload instead of an empty probe:
```bash
./target/release/udp 192.168.1.1 53,123 --payload "TEST_PAYLOAD"
```
*Note: `--payload` can be combined with custom/default ports merged (examples 1 through 3), but **cannot** be combined with `--scout` or `--protocols`.*

#### 5. Timeout Adjusted UDP Scan
Increase timeout to `1500ms` for high-latency/congested networks:
```bash
./target/release/udp 192.168.1.1 53,123 --timeout-ms 1500
```
*Note: `--timeout-ms` can be combined with **any** other scan option.*

#### 6. Concurrency Adjusted UDP Scan
Throttle concurrency to 100 simultaneous probes to protect slower systems or routers:
```bash
./target/release/udp 192.168.1.1 53,123 --limit 100
```
*Note: `--limit` can be combined with **any** other scan option.*

#### 7. Active Protocol Scouting (All 9 Protocols)
Sweep custom target ports sending all registered UDP protocol payloads:
```bash
./target/release/udp 192.168.1.1 53,123,161 --scout
```
*Note: Active scouting sweeps can be combined with custom/default port configurations (examples 1 through 3), timeouts, and concurrency limit overrides.*

#### 8. Specific Protocol Scouting (Subset of Protocols)
Scout custom target ports, sending only specific protocol payloads (e.g., DNS and NTP):
```bash
./target/release/udp 192.168.1.1 53,123 --protocols dns,ntp
```
*Note: The `--protocols` option automatically enables scouting. It can be combined with custom/default port configurations, timeouts, and concurrency limit overrides.*

#### 9. Active Scouting with CSV Export
Scout target ports and write the service mapping results directly to a report file:
```bash
./target/release/udp 192.168.1.1 53,123 --scout --write reports/udp_scout.csv
```
*Note: `--write` can be combined with **any** scanning combination (examples 1 through 8).*

---

## 📡 Layer 2 Discovery Engine (`discover`)

The `discover` tool sweeps local subnets using **ARP** (IPv4) and **NDP** (IPv6) and sniff packet streams to populate the network hosts database.

### CLI Commands & Options

```
Usage: discover [options]
```

| Option | Short | Long | Type | Description |
| :--- | :--- | :--- | :--- | :--- |
| Interface | - | `--interface` | String | Specific interface to bind (e.g. `eth0`, `wlan0`). If omitted, auto-prompts. |
| Timeout | - | `--timeout` | u64 | Seconds to listen for passive and active responses. (Default: `3s`). |
| Custom CIDR | - | `--cidr` | String | Manually target a specific CIDR block (e.g., `10.0.0.0/24` or `fe80::/112`). |
| JSON Output | - | `--json` | Flag | Generates a formatted `discovered_hosts.json` report in the root. |
| Print stdout | - | `--stdout` | Flag | Prints the raw serialized payload structure directly to standard output. |

*Note: By default, if `--stdout` is omitted, the tool automatically saves the aggregated scan findings to `discovered_hosts.csv` in the root workspace directory.*

---

## 🛠 Developer Network Utilities

For developers building additional plugins or scripts within the Bastion ecosystem, the `utils` crate exposes several low-level reusable networking primitives under `utils::net`:

### 1. `UdpFactory`
A generic, chainable builder pattern wrapper for creating standard library (`std::net::UdpSocket`) or async (`tokio::net::UdpSocket`) sockets with customizable options (e.g., timeouts, broadcast mode, and address reuse).
```rust
use utils::net::udp_factory::UdpFactory;

let socket = UdpFactory::new()
    .bind_any(IpAddr::V4(Ipv4Addr::UNSPECIFIED))
    .broadcast(true)
    .build_tokio()
    .await?;
```

### 2. `SsdpScout`
An active service discovery probe executing over UDP multicast. It formats M-SEARCH queries and parses returned device location descriptors and server banners.
```rust
use utils::net::ssdp_scout::SsdpScout;
use std::time::Duration;

let scout = SsdpScout::new("ssdp:all", Duration::from_secs(3))
    .with_bind_ip(local_ip);
let devices = scout.scan().await?;
```

### 3. `KnockSequenceValidator`
A state-tracking engine for building UDP port-knocking services. Tracks individual client progress against sequence combinations across time-bounded windows.
```rust
use utils::net::port_knocker::KnockSequenceValidator;
use std::time::Duration;

let mut validator = KnockSequenceValidator::new(vec![11000, 12000, 13000], Duration::from_secs(3));
let is_complete = validator.register_knock(client_ip, knocked_port);
```

---

## 📦 Standalone Binary Compilation & Distribution

To compile optimized, standalone binaries that can be distributed to systems that do not have `cargo` or the Rust toolchain installed:

1. **Build optimized release binaries**:
   ```bash
   cargo build --release
   ```

2. **Locate the binaries**:
   The standalone binaries are generated under the `target/release/` directory:
   * `target/release/scan`
   * `target/release/udp`
   * `target/release/discover`

---
&copy; 2026. All Rights Reserved. This project is proprietary and closed-source.
