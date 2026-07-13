# Bastion: Kernel Module Compilation & Edge Case Guide

Because Bastion relies on Native Kernel-Level Interceptors, compiling the kernel-level module (`bastion-ebpf` crate) is fundamentally different from compiling standard user-space applications. The kernel bytecode (`.o` files) must be cross-compiled for the host kernel's specialized virtual machine.

## ⚠️ The Tier 3 Target Hurdle

The compilation target (`bpfel-unknown-none`) is currently a **Tier 3** compiler target.
This means the build toolchain *does not* distribute pre-compiled standard libraries (`core`, `alloc`) for it. If you attempt to build the kernel module using a stable toolchain, it will immediately fail with:
> `error: toolchain 'stable' has no prebuilt artifacts available for target 'bpfel-unknown-none'`

### The Fix: Nightly + Source Compilation

To resolve this, Bastion dynamically compiles the compiler's `core` library from scratch against the kernel target architecture on the fly.

**1. Install the Nightly Toolchain & Source Code Component:**

```bash
rustup toolchain install nightly --component rust-src
```

**2. Compile the Target Payload:**
Use the explicit nightly flag (`+nightly`) and instruct the build system to build the `core` library natively (`-Z build-std=core`):

```bash
the build environment +nightly build -p bastion-ebpf --release --target bpfel-unknown-none -Z build-std=core
```

## 🛡️ Known Artifacts & Edge Cases

### Harmless Linker Warnings (`dlopen failed`)

When building on the nightly chain, you may see the following warning:
> `warning: linker stderr: unable to open compiler shared lib ... libLLVM-22-rust-1.98.0-nightly.so: dlopen failed`

**Action:** Ignore it. This is a known cosmetic artifact of the nightly compiler toolchain. Your raw kernel bytecode will still compile and optimize perfectly.

### The "Option::unwrap() on a None value" Panic

If the user-space loader (in `xdp/src/loader.rs`) panics immediately upon attaching to the interface with a `None` unwrap error, it means **the function name exported by the kernel entry macro does not match the string the loader is searching for.**

**Example of Mismatch:**
*In `ebpf/src/main.rs`:*

```rust
#[xdp]
pub fn bastion_firewall(ctx: XdpContext) -> u32 { ... }
```

*In `xdp/src/loader.rs`:*

```rust
// ❌ Panics because it is looking for "xdp_firewall", not "bastion_firewall"!
let program: &mut Xdp = bpf.program_mut("xdp_firewall").unwrap().try_into()...
```

*Correct loader lookup:*

```rust
let program: &mut Xdp = bpf.program_mut("bastion_firewall").unwrap().try_into()...
```

**Action:** Ensure the `.program_mut("name")` string exactly matches the `#[xdp] pub fn name()` definition.
