# Recon & Forge Architecture

This document outlines the architecture of the **Recon** and **Forge** crates added to Bastion. These represent the next evolution in the Bastion framework, advancing past simple passive eBPF blocking and WiFi monitoring into active, pro-grade network mapping and stateless packet manipulation.

## 1. The `forge` Crate (Packet Crafting & Raw Sockets)

The `forge` crate is the cornerstone of stateless network interactions. It completely bypasses the host operating system's TCP/IP stack to provide $O(1)$ raw packet injection and crafting capabilities.

### Forge Key Capabilities

- **`RawSocketFactory` ([source](../forge/src/socket/raw.rs))**: Provides native `#![cfg(target_os = "linux")]` implementations for `IP_HDRINCL` raw sockets (`SOCK_RAW`) using pure `libc` FFI and `std::os::fd::AsRawFd`. This allows Bastion to operate as a high-speed blaster without suffering thread-blocking context switches.
- **`TcpScoutFactory` ([source](../forge/src/packet/tcp.rs))**: Hand-rolled native TCP packet crafting factories via `etherparse`. Ensures granular control over flags (e.g. `SYN`), sequence numbers, and explicitly rejects edge cases like zero-windows for valid scout probes.
- **`UdpScoutFactory` ([source](../forge/src/packet/udp.rs))**: Comprehensive UDP packet forging capable of arbitrary payload injection natively onto the wire, capped accurately at the UDP 65507 byte limit.

## 2. The `recon` Crate (Stateless Active Scanning)

The `recon` crate bridges active discovery protocols into Bastion using the heavy-lifting provided by `forge`.

### Recon Key Capabilities

- **Stateless Scanning ([sweep/stateless.rs](../recon/src/sweep/stateless.rs))**: Emits `SYN` bursts blindly across subnets utilizing `forge::RawSocketFactory`. Completely decoupled sending (Tx) and receiving (Rx) threads for asynchronous blast radius coverage.
- **OS Fingerprinting ([fingerprint/os.rs](../recon/src/fingerprint/os.rs))**: Passive OS detection via TCP/IP heuristic analysis (TTL, Window Size). Successfully identifies Windows, Linux kernels, macOS, iOS, Solaris, recognizing intentional vs malformed TTL anomalies dynamically.

## 3. Dynamic OUI Resolution ([utils::net::oui](../utils/src/net/oui.rs))

Added to the core `utils`, Bastion now features live dynamic MAC address vendor resolution, retiring the outdated, hardcoded `.csv` dependencies. It natively fetches IEEE OUI records and updates an in-memory cached lookup table.

## The Architectural Win

By migrating active mitigation and stateless mapping to `forge`/`recon`, Bastion eliminates the bottleneck of standard socket APIs (`std::net::TcpStream`). All tasks are fully TDD proven, isolating edge cases before they touch the network interface.

---
&copy; 2026. All Rights Reserved. This project is proprietary and closed-source.
