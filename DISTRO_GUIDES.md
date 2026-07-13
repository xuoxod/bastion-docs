# Quick Guide: What to Expect from Linux Deployments

Bastion is specifically tailored for hardened production Linux environments. This document covers practical tips, behavioral expectations, and common edge cases.

---

## 🐧 Distro-Specific Checklists

### **1. Ubuntu / Debian (`apt`)**

*What to expect:* Extremely stable, standard usage.

- Standard capabilities work exactly as described.
- UFW is almost always active.
- *Pro tip*: If `bastion-daemon` appears restricted, double-check that your `apparmor` profiles aren't blocking raw `bpf()` syscalls for non-kernel binaries.

### **2. RHEL / CentOS / AlmaLinux (`dnf`)**

*What to expect:* SELinux enforcement.

- **SELinux** operates in `Enforcing` mode by default. Because Bastion mounts to the networking interface via XDP, you must explicitly flag the binary in your policy:

  ```bash
  # Allow the binary to manage network packets directly
  sudo semanage permissive -a bastion_t 
  ```

- *Pro tip:* Use `restorecon -v /usr/local/bin/bastion-daemon` if you notice permissions errors launching via `systemctl`.

### **3. Alpine Linux (Docker / K8s Nodes)**

*What to expect:* Musl-libc dependencies.

- You must compile Bastion statically using `musl` if deploying directly onto the bare metal of an Alpine instance.
- *Pro tip:* `cargo build --release --target x86_64-unknown-linux-musl` will produce an identical binary size without strictly missing the `glibc` library on start.

---

## 💻 Expected Console Outcomes

When running the systemd installer (`scripts/systemd/install_service.sh`) or the CLI tools, this is the expected operational flow on a healthy environment. Notice standard info/success prefixes:

```text
[INFO] Provisioning configuration directories natively in /etc/bastion
[INFO] Checking for local compiled binaries...
[SUCCESS] Copied target/release/bastion-daemon -> /usr/local/bin
[INFO] Installing the hardened Systemctl unit...
[SUCCESS] Installed unit to /etc/systemd/system/bastion.service
[INFO] Reloading systemd daemon to ingest exact capabilities...
[INFO] Enabling Bastion to boot perfectly on startup...
[INFO] Checking firewall environment...
[SUCCESS] UFW is active! Bastion's XDP natively operates under UFW seamlessly. No conflicts!
```

If it executes properly, using `systemctl status bastion` will yield the active listening state:

```text
● bastion.service - Bastion Core eBPF Firewall Daemon
     Loaded: loaded (/etc/systemd/system/bastion.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-04-20 09:12:00 UTC; 5min ago
       Docs: https://github.com/xuoxod/bastion
   Main PID: 104521 (bastion-daemon)
      Tasks: 2 (limit: 38210)
     Memory: 1.8M
     CGroup: /system.slice/bastion.service
             └─104521 /usr/local/bin/bastion-daemon --iface eth0

Apr 20 09:12:00 node01 bastion-daemon[104521]: [INFO] BPF Interface Bound: eth0 (XDP_MODE_NATIVE)
Apr 20 09:12:00 node01 bastion-daemon[104521]: [INFO] Lock-free atomic RCU pointer online.
```

---

## 🔍 Advanced Troubleshooting & Details

### **How to verify Bastion eBPF hooks using pure Linux tools**

Bastion modifies the kernel memory using native OS APIs. If you want to verify that Bastion is *actually* active beneath the application layer without reading Bastion's logs, use native `ip` routing tools.

```bash
# Ask the kernel to show what is attached to eth0 natively
ip link show eth0
```

**Expected output:**

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 xdp qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
    prog/xdp id 137 tag 6f001234abcd
```

Look for **`prog/xdp id [ID]`** inside the `ip link` dump. If that is present on the interface, Bastion has seized control of the hardware queue.

### **Reading The Raw eBPF Map Values Natively**

Curious if the C-ABI integration from `netscan-bridge` worked? Rather than running tests within Go, query the exact memory structure using `bpftool`:

```bash
# Dump the exact elements inside the eBPF memory map without stopping the firewall
sudo bpftool map dump name IP_BLOCKLIST
```

*Expected output (Obfuscated, typical Class-C blocks):*

```c
key:
c0 a8 01 32  // (192.168.1.50 in u32 hex)
value:
01           // (Blocked boolean = 1)
```

---

### 🔥 Summary Checklist

- [x] Compilation finished via `release` (or `musl` for Alpine).
- [x] Installer triggered via `sudo`.
- [x] Target Network Device verifies `xdp` binding.
- [x] Test-bed integrations leverage `SIGHUP` and pointer RCU efficiently.

---
&copy; 2026. All Rights Reserved. This project is proprietary and closed-source.
