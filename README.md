# 🛡️ Bastion: Enterprise Native Network Assessment & Interdiction Engine

Welcome to the **Bastion** documentation and architecture showcase repository. This repository serves as the public-facing architectural blueprint, topology mapping, and operational reference for the closed-source `bastion` project.

Bastion is a highly modular, enterprise-grade native compiled system network assessment and defense engine. It features zero-allocation binary parsing, asynchronous raw socket sniffing, and lock-free kernel-level packet filter hooks.

---

## 🏗️ System Architecture & Data Flow

Bastion is built as a capability-first background daemon paired with CLI controller planes and native library boundaries:

```mermaid
flowchart TB
    subgraph User Interface Plane
        A[Dashboard Controller] -- WebSockets / Signaling -- > B[RMediaTech Portal]
        C[CLI Utility Plane] -- Unix Domain Sockets UDS --> D[Bastion Daemon]
    end

    subgraph Process Plane
        B -- Coordination Bridge --> D
        D -- Asynchronous Process Spawn --> E[High-Speed Scanner 'scan']
        D -- Asynchronous Process Spawn --> F[UDP Scout Engine 'udp']
        D -- Asynchronous Process Spawn --> G[BLE Sweep Engine 'ble_scan']
    end

    subgraph Kernel & Hardware Plane
        D -- Packet Forge --> H[Raw Socket Packet Injector]
        D -- Sniff/Analyze --> I[eBPF / XDP Socket Filter]
        H --> J[Physical Interface Layer]
        I --> J
    end

    style D fill:#0b3d2e,stroke:#27ae60,color:#d5ffe9,stroke-width:1px
    style I fill:#1f3c4d,stroke:#3498db,color:#d5f0ff,stroke-width:1px
    style E fill:#1e1e1e,stroke:#333,color:#fff
    style F fill:#1e1e1e,stroke:#333,color:#fff
    style G fill:#1e1e1e,stroke:#333,color:#fff
```

---

## 🧩 Workspace Crate Architecture

Bastion strictly adheres to Separation of Concerns via modular Rust workspaces:

* **`daemon`**: The high-throughput background coordinator. Exposes local Unix Domain Sockets (UDS) and establishes dynamic secure WebSocket coordinate links to the signaling mesh. It manages active sub-processes asynchronously using a thread-safe coordinator, allowing instantaneous process aborts.
* **`cli`**: The command-line control utility. Communicates with the background daemon over UDS to trigger operations and query subsystem status (`ping`, `status`, `block`, `unblock`).
* **`utils`**: The utility library containing CIDR engines, network-card self-awareness checks, and the high-speed port mapping engine (`scan`).
* **`forge`**: A zero-copy TCP/UDP raw packet factory with O(1) buffer allocation.
* **`recon`**: Active/passive service fingerprinting and passive wire-sniffing. Ingests TTL and Hop Limit metrics off the wire for zero-probe OS classification.
* **`strike`**: Active wireless interdiction and Deauth injection for supported radio adapters.
* **`ebpf` & `xdp`**: The raw kernel-level packet hook and BPF maps layer, allowing high-performance traffic block and unblock controls directly in the network card driver path.
* **`ffi`**: The stable C-compatible native interface (`libbastion_ffi.so`), allowing foreign runtime integrations.

---

## 🛡️ Core Capabilities

### 1. Zero-Probe Passive OS Fingerprinting
By monitoring network interfaces passively, Bastion intercepts TCP syn-acks, matching IPv4 TTLs, window sizes, and IPv6 Hop Limits against known operating system signature profiles. This allows it to classify network hosts without sending a single active probe.

### 2. Lock-Free Packet Mitigation (eBPF/XDP)
Utilizes eBPF bytecode loaded directly into the kernel network hook points. When the operator issues a block/unblock policy, the system updates a shared lock-free BPF map, allowing packet drop evaluations to complete in nanoseconds at the driver level.

### 3. Non-Blocking WebSockets Orchestration
The daemon maintains a persistent coordinate connection with the central dashboard, streaming structured CSV and JSON outputs in real-time. Command execution is delegated to spawned asynchronous readers, preventing the coordination connection from blocking during active sweeps.

---

## 🕶️ Operational Disclosure Model

Documentation in this repository emphasizes operational outcomes, interfaces, and architecture topologies over vendor-specific deployment details. Low-level internal structures are mapped only where required for platform compatibility.

---

&copy; 2026. All Rights Reserved. This showcase documentation is open-source; the core implementation codebase is private and proprietary.
