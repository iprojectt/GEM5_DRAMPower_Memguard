---

# GEM5 Setup and Build Guide (Chronological Steps)

## Overview

This document describes the sequence of steps performed to successfully build and configure **gem5 v20.1.0.4** for running **SPEC CPU2017 workloads** with **DRAM tracing** and **MemGuard experiments**.

Target environment used:

| Component | Version |
|---|---|
| OS | Ubuntu 22.04.5 LTS |
| Python | 3.10.12 |
| Compiler | g++-9 |
| glibc | 2.35 |
| gem5 | v20.1.0.4 |

Each step is listed in the exact order it was executed.

---

# Step 1: Obtain gem5 Source

Download or clone the gem5 source code and place it inside the project directory.

Example project layout:

```text
spec_2017/
│
├── configs/
├── disk-image/
├── gem5/
├── gem5-resources/
├── results/
├── linux-4.19.83-bzImage
├── process_trace.sh
├── run_single_benchmark_intr_term.sh
├── run_single_benchmark.sh
├── spec-initrd.img
└── vmlinux-4.19.83
```

Navigate to the gem5 root directory:

```bash
cd gem5
```

---

# Step 2: Install Required Build Dependencies

Install required packages:

```bash
sudo apt update

sudo apt install \
    python-is-python3 \
    python3-pip \
    g++-9 \
    m4
```

---

# Step 3: Install Compatible SCons Version

This step was required because of  
→ [[#Problem 1: SCons Version Compatibility]]

Install the compatible version:

```bash
pip3 install --user scons==3.1.2 six
```

Add local binaries to PATH:

```bash
export PATH=$HOME/.local/bin:$PATH
```

Verify:

```bash
scons --version
```

---

# Step 4: Patch Python Compatibility Issue

This step was required because of  
→ [[#Problem 2: Python 3.10 collections.Mapping Error]]

Patch the source file:

```bash
sed -i 's/from collections import Mapping/from collections.abc import Mapping/g' \
src/python/m5/debug.py
```

---

# Step 5: Patch gem5 Source Code

This step was required because of  
→ [[#Problem 3: SIGSTKSZ Compilation Error]]

Open the file:

```text
src/sim/init_signals.cc
```

Replace:

```cpp
static uint8_t fatalSigStack[2 * SIGSTKSZ];
```

With:

```cpp
static uint8_t *fatalSigStack = new uint8_t[2 * SIGSTKSZ];
```

---

# Step 6: Build gem5

Remove old builds:

```bash
rm -rf build
```

Compile gem5:

```bash
scons build/X86/gem5.opt -j$(nproc) \
    PYTHON_CONFIG=/usr/bin/python3-config \
    CXX=g++-9
```

---

# Step 7: Verify gem5 Build

Verify the gem5 binary runs:

```bash
build/X86/gem5.opt --help
```

If successful, gem5 displays its usage information.

---

# Step 8: Build gem5 Utility Programs

Build the m5 utility:

```bash
cd util/m5
scons build/x86/out/m5
```

Build terminal utility:

```bash
cd ../term
make
```

This produces:

```text
m5term
```

---

# Step 9: Enable KVM Performance Counters

This step was required because of  
→ [[#Problem 4: KVM Performance Counter Permission Error]]

Temporary fix:

```bash
sudo sh -c 'echo -1 > /proc/sys/kernel/perf_event_paranoid'
```

Permanent fix:

Edit:

```text
/etc/sysctl.conf
```

Add:

```text
kernel.perf_event_paranoid=-1
```

Apply:

```bash
sudo sysctl -p
```

Verify:

```bash
cat /proc/sys/kernel/perf_event_paranoid
```

Expected output:

```text
-1
```

---

# Step 10: Run gem5 Simulation

Run the benchmark script:

```bash
./run_single_benchmark.sh
```

This performs:

1. Boot Linux using KVM CPUs
2. Fast-forward workload execution
3. Switch to O3 CPUs at ROI
4. Run detailed simulation
5. Generate DRAM traces

---

# Output Files

Simulation output is generated in:

```text
results/<cpu>/<size>/<benchmark>_dramtrace/
```

Important files include:

| File | Description |
|---|---|
| dram_raw_trace.txt | DRAM state trace |
| stats.txt | gem5 statistics |
| simout | simulation output |
| simerr | simulation errors |

---

# Execution Pipeline

```text
Host Machine
     │
     ▼
gem5 Simulator
     │
     ▼
Guest Linux Kernel
     │
     ▼
KVM Fast Forward
     │
     ▼
ROI CPU Switch (O3)
     │
     ▼
SPEC CPU2017 Workloads
     │
     ▼
DRAM State Trace
     │
     ▼
DRAMPower Analysis
```

---

# Referenced Problems

---

## Problem 1: SCons Version Compatibility

Newer SCons versions break gem5 v20 builds.

Fix applied in:

→ [[#Step 3: Install Compatible SCons Version]]

---

## Problem 2: Python 3.10 collections.Mapping Error

Error observed:

```text
ImportError: cannot import name 'Mapping' from 'collections'
```

Cause:

`collections.Mapping` was moved to:

```python
collections.abc.Mapping
```

in Python ≥ 3.10.

Fix applied in:

→ [[#Step 4: Patch Python Compatibility Issue]]

---

## Problem 3: SIGSTKSZ Compilation Error

Error observed:

```text
array bound is not an integer constant before ']'
```

Fix applied in:

→ [[#Step 5: Patch gem5 Source Code]]

---

## Problem 4: KVM Performance Counter Permission Error

Error observed:

```text
PerfKvmCounter::attach received error EACCESS
```

Cause:

Linux kernel perf restrictions prevented gem5 KVM CPUs from accessing performance counters during KVM fast-forwarding.

Fix applied in:

→ [[#Step 9: Enable KVM Performance Counters]]

---
