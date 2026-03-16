
---

# GEM5 Setup and Build Guide (Chronological Steps)

## Overview

This document describes the sequence of steps performed to successfully build and configure **gem5 v20.1.0.4** for running **SPEC CPU2017 workloads** with **DRAM tracing** and **MemGuard experiments**.

Each step is listed in the **exact order it was executed**.  
If a step exists because of a specific issue, the step links to the corresponding **problem description**.

---

# Step 1: Obtain gem5 Source

Download or clone the gem5 source code and place it inside the project directory.

Example project layout:

```
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

```
cd gem5
```

---

# Step 2: Create Python 3.8 Virtual Environment

This step was required because of  
→ [[#Problem 1: Python Version Compatibility Issue]]

Create the environment:

```bash
python3.8 -m venv gem5_py38_env
```

Activate it:

```bash
source gem5_py38_env/bin/activate
```

Verify:

```bash
python --version
```

---

# Step 3: Install Compatible SCons Version

This step was required because of  
→ [[#Problem 2: SCons Version Compatibility]]

Install the compatible version:

```bash
pip install scons==3.1.2
```

Verify:

```bash
scons --version
```

---

# Step 4: Install Required Python Dependencies

This step was required because of  
→ [[#Problem 3: Missing Python Module six]]

Install the dependency:

```bash
pip install six
```

Also install for the Python environment used internally by gem5:

```bash
~/.pyenv/versions/3.8.17/bin/pip install six
```

---

# Step 5: Install Compatible Compiler

This step was required because of  
→ [[#Problem 4: Compiler Compatibility]]

Install GCC 9:

```bash
sudo apt install g++-9
```

---

# Step 6: Patch gem5 Source Code

This step was required because of  
→ [[#Problem 5: SIGSTKSZ Compilation Error]]

Open the file:

```
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

# Step 7: Build gem5

This step depends on fixes applied in:

- [[#Step 2: Create Python 3.8 Virtual Environment]]
    
- [[#Step 3: Install Compatible SCons Version]]
    
- [[#Step 6: Patch gem5 Source Code]]
    

Remove old builds:

```bash
rm -rf build
```

Compile gem5:

```bash
scons build/X86/gem5.opt -j$(nproc) CXX=g++-9
```

---

# Step 8: Verify gem5 Build

Verify the gem5 binary runs:

```bash
build/X86/gem5.opt --help
```

If successful, gem5 displays its usage information.

---

# Step 9: Build gem5 Utility Programs

Build the **m5 utility**.

```
cd gem5/util/m5
```

Compile:

```bash
scons build/x86/out/m5
```

---

# Step 10: Build Terminal Utility

Build the terminal program used for guest interaction.

```
cd gem5/util/term
```

Compile:

```bash
make
```

This produces:

```
m5term
```

---

# Step 11: Enable KVM Performance Counters

This step was required because of  
→ [[#Problem 6: KVM Performance Counter Permission Error]]

Run:

```bash
sudo sysctl -w kernel.perf_event_paranoid=0
```

Make permanent:

Edit:

```
/etc/sysctl.conf
```

Add:

```
kernel.perf_event_paranoid = 0
```

Apply:

```bash
sudo sysctl -p
```

---

# Step 12: Run gem5 Simulation

Run the benchmark script:

```bash
./run_single_benchmark.sh
```

This performs:

1. Boot gem5
    
2. Load Linux kernel
    
3. Mount SPEC disk image
    
4. Run SPEC workload
    
5. Generate DRAM trace
    

---

# Output Files

Simulation output is generated in:

```
results/<cpu>/<size>/<benchmark>_dramtrace/
```

Important files include:

| File               | Description       |
| ------------------ | ----------------- |
| dram_raw_trace.txt | DRAM state trace  |
| stats.txt          | gem5 statistics   |
| simout             | simulation output |
| simerr             | simulation errors |

---

# Execution Pipeline

```
Host Machine
     │
     ▼
gem5 Simulator
     │
     ▼
Guest Linux Kernel
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

The following problems explain **why certain steps were required**.

---

## Problem 1: Python Version Compatibility Issue

Error observed:

```
ModuleNotFoundError: No module named 'imp'
```

Cause:

The `imp` module was removed in Python ≥ 3.10.

Fix applied in:

→ [[#Step 2: Create Python 3.8 Virtual Environment]]

---

## Problem 2: SCons Version Compatibility

Newer SCons versions break gem5 v20 builds.

Fix applied in:

→ [[#Step 3: Install Compatible SCons Version]]

---

## Problem 3: Missing Python Module six

Error:

```
ModuleNotFoundError: six
```

Fix applied in:

→ [[#Step 4: Install Required Python Dependencies]]

---

## Problem 4: Compiler Compatibility

Modern GCC versions caused build errors.

Fix applied in:

→ [[#Step 5: Install Compatible Compiler]]

---

## Problem 5: SIGSTKSZ Compilation Error

Error:

```
array bound is not an integer constant before ']'
```

Fix applied in:

→ [[#Step 6: Patch gem5 Source Code]]

---

## Problem 6: KVM Performance Counter Permission Error

Error:

```
PerfKvmCounter::attach received error EACCESS
```

Fix applied in:

→ [[#Step 11: Enable KVM Performance Counters]]

---


