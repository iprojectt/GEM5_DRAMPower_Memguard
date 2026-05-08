# GEM5 Setup and Build Guide

## Overview

This document describes the sequence of steps performed to successfully build and configure **gem5 v20.1.0.4** for running **SPEC CPU2017 workloads** with **DRAM tracing** and **MemGuard experiments**.

### Target Environment

| Component | Version |
|---|---|
| OS | Ubuntu 22.04.5 LTS |
| Python | 3.10.12 |
| Compiler | g++-9 |
| glibc | 2.35 |
| gem5 | v20.1.0.4 |

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

Newer SCons versions are incompatible with gem5 v20.1.0.4.

Install compatible SCons and Python dependency:

```bash
pip3 install --user scons==3.1.2 six
```

Add local binaries to PATH:

```bash
export PATH=$HOME/.local/bin:$PATH
```

Verify installation:

```bash
scons --version
```

---

# Step 4: Patch Python 3.10 Compatibility Issue

gem5 v20.1.0.4 contains an old Python import that is incompatible with Python 3.10.

Patch the source file:

```bash
sed -i 's/from collections import Mapping/from collections.abc import Mapping/g' \
src/python/m5/debug.py
```

---

# Step 5: Patch SIGSTKSZ Compilation Issue

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

If successful, gem5 displays usage information.

---

# Step 8: Build gem5 Utility Programs

Build the m5 utility:

```bash
cd util/m5
scons build/x86/out/m5
```

Build the terminal utility:

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

This step is required for KVM fast-forwarding and ROI CPU switching workflows.

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

Apply changes:

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

This workflow performs:

1. Linux boot using KVM CPUs
2. Fast-forwarding workload execution
3. CPU switch to O3 at ROI
4. Detailed simulation
5. DRAM trace generation

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
| simout | Simulation output |
| simerr | Simulation errors |

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

# Common Problems and Fixes

## Problem 1: SCons Compatibility Issue

### Error

```text
scons build failures with newer SCons versions
```

### Cause

gem5 v20.1.0.4 is incompatible with newer SCons releases.

### Fix

```bash
pip3 install --user scons==3.1.2
```

---

## Problem 2: Python 3.10 Mapping Import Error

### Error

```text
ImportError: cannot import name 'Mapping' from 'collections'
```

### Cause

Python 3.10 moved `Mapping` to:

```python
collections.abc.Mapping
```

### Fix

```bash
sed -i 's/from collections import Mapping/from collections.abc import Mapping/g' \
src/python/m5/debug.py
```

---

## Problem 3: SIGSTKSZ Compilation Error

### Error

```text
array bound is not an integer constant before ']'
```

### Cause

Modern compilers reject the original static allocation.

### Fix

Replace:

```cpp
static uint8_t fatalSigStack[2 * SIGSTKSZ];
```

With:

```cpp
static uint8_t *fatalSigStack = new uint8_t[2 * SIGSTKSZ];
```

---

## Problem 4: KVM Performance Counter Permission Error

### Error

```text
PerfKvmCounter::attach received error EACCESS
```

### Cause

Linux perf restrictions block KVM performance counter access.

### Fix

```bash
sudo sh -c 'echo -1 > /proc/sys/kernel/perf_event_paranoid'
```

Permanent fix:

```text
kernel.perf_event_paranoid=-1
```

inside:

```text
/etc/sysctl.conf
```

---
