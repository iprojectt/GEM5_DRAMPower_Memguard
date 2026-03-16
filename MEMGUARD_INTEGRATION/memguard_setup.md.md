
---

# MemGuard Kernel Module Setup in gem5

## Overview

MemGuard is a Linux kernel module used to **control and enforce memory bandwidth limits** for CPU cores.  In this setup, MemGuard is compiled and loaded inside a **Linux guest running in gem5**.

The purpose of integrating MemGuard is to enable **memory bandwidth control experiments** during SPEC benchmark execution.

This document describes:

- Preparing the kernel environment
    
- Building the MemGuard module
    
- Loading the module inside the gem5 guest
    
- Issues encountered and fixes applied
    

---

# System Context

The MemGuard module is compiled **inside the gem5 guest system** using the same kernel source that the guest OS was built from.

Kernel used:

```
Linux 4.19.83
```

Kernel source directory inside the guest: 
# Vedant yaha add karega ki how to get this directory inside diskimage

```
/home/gem5/linux-4.19.83
```

MemGuard module location:

# Yaha par git clone karke memguard download kiye the

```
/home/gem5/kernel-mods/memguard
```

---


# Issue Encountered: Invalid Module Format

### Error

When tried to run inside the Interactive Terminal inside GEM5 FS simulation. 
# Yaha link add kardo Debug(interactive GEM5()) page ka

```
insmod memguard.ko
```


During module insertion the following error may occur:


```
Invalid module format
```

### Cause

The module was compiled against a kernel source tree that had not yet generated the required symbol information.

### Fix

Ensure the kernel build environment has been prepared correctly.
Run the following commands in the kernel source directory:

```bash
make prepare
make scripts
make modules_prepare
make -j2
```

Then rebuild the module. I will talk about these Later.

---




# Step 1: Boot gem5 and Obtain Shell Access

Boot the gem5 system.

```

qemu-system-x86_64 \
  -enable-kvm \
  -cpu host,pmu=on \
  -m 8G \
  -smp 4 \
  -kernel ~/Desktop/spec_project/spec_2017/linux-4.19.83-bzImage \
  -initrd ~/Desktop/spec_project/spec_2017/spec-initrd.img \
  -append "root=/dev/sda1 rw console=ttyS0 nokaslr" \
  -drive file=/home/vedanttejas/Desktop/spec_project/spec_2017/disk-image/spec-2017/spec-2017-image/spec-2017-eee,format=raw \
  -net user,hostfwd=tcp::2222-:22 \
  -net nic \
  -nographic

```

# THIS ONE I AM TALKING ABOUT TO OPEN WTH QEMU, so THOSE TWO FILES YOU SHOULD ENTION AND HOW TO CREATE THEM.



ALSO KIND OF TRY TO DO WITH DEFAULT HEADER AND COMPILING FOR THAT CONGIFX86... and checking IF IT WORK THAT WAY WITHOUT USING SPECIFIC 4.19 in QEMU

---

# Step 2: Navigate to Kernel Source

Inside the guest system:

```bash
cd /home/gem5/linux-4.19.83
```

---

# Step 3: Copy gem5 Kernel Configuration

Copy the kernel configuration used for the gem5 kernel build.

```bash
cp /home/gem5/config.4.19.83 .config
```

This ensures the kernel source tree uses the same configuration as the running kernel.

---

# Step 4: Update Kernel Configuration

Synchronize configuration options with the kernel source:

```bash
make oldconfig
```

This updates the configuration file to match the kernel source tree.

---

# Step 5: Prepare Kernel Build Environment

Prepare kernel headers required for module compilation.

```bash
make prepare
```

---

# Step 6: Build Kernel Scripts

Kernel module builds require helper scripts such as `modpost`.

Build these scripts:

```bash
make scripts
```

---

# Step 7: Prepare Module Build Environment

Generate additional headers required for external modules.

```bash
make modules_prepare
```

---

# Step 8: Compile the Kernel (Required Once)

Compile the kernel once to generate symbol version information.

```bash
make -j2
```

This step generates important files required for module compilation:

```
Module.symvers
generated kernel headers
symbol version metadata
```

Without these files, kernel modules may fail to load.

---

# NOW AGAIN INSIDE INTERACTIVE GEM5 TERMINAL


# Step 9: Navigate to MemGuard Module Source

Move to the MemGuard source directory.

```bash
cd /home/gem5/kernel-mods/memguard
```

---

# Step 10: Clean Previous Builds

Remove previous module build artifacts.

```bash
make clean
```

---

# Step 11: Compile MemGuard Module

Compile the MemGuard module.

```bash
make
```

Successful compilation produces:

```
memguard.ko
```

---

# Step 12: Load MemGuard Module

Insert the kernel module into the running kernel.

```bash
insmod memguard.ko
```

---

# Step 13: Verify Module Loaded

Check whether the module is active.

```bash
lsmod | grep memguard
```

Expected output:

```
memguard
```

---



Once the guest OS is running, connect to the guest console using:

```bash
gem5/util/term/m5term localhost 3456
```

Login to the system as root.





# Issue Encountered: PMU Initialization Failure

MemGuard may attempt to initialize hardware performance counters.

However, gem5 in KVM mode may restrict access to PMU features.

If PMU initialization fails, the module may need to be modified to disable PMU-based monitoring and rely on alternative bandwidth tracking methods.

---

# Directory Layout

Relevant directories inside the guest system:

```
/home/gem5
│
├── linux-4.19.83
│   └── kernel source
│
├── kernel-mods
│   └── memguard
│        ├── Makefile
│        ├── memguard.c
│        └── memguard.ko
│
└── config.4.19.83
```

---

# MemGuard Integration Pipeline

```
Host Machine
     │
     ▼
gem5 Simulator
     │
     ▼
Guest Linux Kernel (4.19.83)
     │
     ▼
MemGuard Kernel Module
     │
     ▼
Memory Bandwidth Control
     │
     ▼
SPEC Benchmark Execution
```

---

# Summary

MemGuard was successfully integrated into the gem5 guest system through the following process:

1. Boot gem5 and access guest shell
    
2. Prepare kernel build environment
    
3. Compile the Linux kernel
    
4. Compile MemGuard module
    
5. Insert module into running kernel
    

This setup enables memory bandwidth control experiments within the gem5 simulation environment.

---

If you want, I can also help you create **three more Obsidian notes** that make your documentation **much cleaner and research-grade**:

1️⃣ **gem5_build_setup.md** (all gem5 build issues + fixes)  
2️⃣ **spec_boot_pipeline.md** (how SPEC runs inside gem5)  
3️⃣ **system_architecture.md** (full system diagram + pipeline)

These four notes together will form a **perfect reproducibility documentation set.**