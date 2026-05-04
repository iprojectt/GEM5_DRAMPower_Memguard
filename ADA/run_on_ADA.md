Here is your content rewritten in **clean, GitHub-ready Markdown**, with **all commands included properly**.

---

# 🔧 ADA Build Environment Analysis for gem5

This section documents the **actual runtime environment on ADA (IIIT cluster)**.
The goal is to **match this environment locally** to avoid build/runtime incompatibilities.

---

# 🎯 Final ADA Environment (Ground Truth)

All values below were obtained directly from an **interactive compute node**.

---

## 🖥 OS

```bash
cat /etc/os-release
```

```text
Ubuntu 22.04.5 LTS
```

---

## 🐍 Python

```bash
python3 --version
which python3
```

```text
Python 3.10.12
/bin/python3
```

Check Python linking configuration:

```bash
python3-config --ldflags
python3-config --includes
```

```text
libpython3.10.so ✔
```

Verify Python shared libraries:

```bash
ldconfig -p | grep libpython
```

---

## 🧱 Compiler

```bash
gcc --version
g++ --version
```

```text
gcc/g++ 11.4.0
```

---

## 🔗 glibc (VERY IMPORTANT)

```bash
ldd --version
```

```text
glibc 2.35
```

---

## 📦 Libraries

Check installed runtime libraries:

```bash
ldconfig -p | grep protobuf
ldconfig -p | grep hdf5
ldconfig -p | grep zlib
```

```text
protobuf ✔
hdf5 ✔
zlib ✔
```

---

## ⚙️ Modules

```bash
which module
```

```text
No module system ❌
```

---

# 🧠 Key Insights

---

## 1. Python Version is a HARD Requirement

```text
You MUST build with Python 3.10
```

* gem5 v20 depends on Python ≤ 3.10
* Mismatch leads to runtime errors (`libpython mismatch`)

---

## 2. Compiler Version is NOT Critical for Runtime

```text
g++-11 exists on ADA → irrelevant for execution
```

* You can safely build using **g++-9 locally**
* Compiler only affects build, not execution

---

## 3. glibc is the MOST IMPORTANT Constraint

```text
ADA glibc = 2.35
```

### Rule:

> Your build system must use **glibc ≤ 2.35**

---

# 🚨 Critical Compatibility Rule

## ❌ Building on newer systems (e.g., Ubuntu 24.04)

```text
glibc 2.39 → incompatible
```

Result:

```text
./gem5.opt: version `GLIBC_2.38' not found
```

💥 Binary will **NOT run on ADA**

---

## ✅ Correct Approach

```text
Build on Ubuntu 22.04
```

✔ Matches ADA exactly
✔ No runtime issues

---

# 🎯 Recommended Build Environment

Use the following configuration:

```text
OS:        Ubuntu 22.04
Python:    3.10.x
Compiler:  g++-9
glibc:     2.35
gem5:      v20.1.0.4
```

---

# 🚀 Final Build Command

```bash
scons build/X86/gem5.opt -j$(nproc) \
    PYTHON_CONFIG=/usr/bin/python3.10-config \
    CXX=g++-9
```

---

# 🔍 Verification Checklist

Before transferring to ADA:

---

## 1. Check linked libraries

```bash
ldd build/X86/gem5.opt
```

Expected:

* ✔ `libpython3.10.so`
* ✔ `/lib/x86_64-linux-gnu/libc.so.6`

---

## 2. Check GLIBC compatibility

```bash
strings build/X86/gem5.opt | grep GLIBC_
```

### ✔ Valid output:

```text
GLIBC_2.34
GLIBC_2.35
```

### ❌ Invalid output:

```text
GLIBC_2.38
GLIBC_2.39
```

---

# 🧠 Final Mental Model

```text
Python → must match exactly
glibc → must not be newer than target
compiler → irrelevant at runtime
```

---

# 🎯 Final Conclusion

> ✅ Build gem5 on **Ubuntu 22.04 with Python 3.10 and g++-9**
> ❌ Do NOT build on Ubuntu 24.04

---

# 🔥 Outcome

Following this setup ensures:

* No Python mismatch errors
* No GLIBC runtime failures
* No compiler incompatibility issues

---
