
and link it from your main setup page with:

```text
→ [[modify_run_spec_for_interactive_boot]]
```

---

# Modify `run_spec.py` for Interactive gem5 Boot

## Purpose

The original `configs/run_spec.py` script automatically launches a SPEC CPU2017 benchmark after the Linux guest finishes booting.  
For debugging and development tasks (such as compiling kernel modules like **MemGuard**), this behavior prevents user interaction with the guest system.

To enable an **interactive full-system (FS) boot**, the script was modified so that:

- SPEC benchmarks are **not automatically executed**
    
- gem5 **keeps running after Linux boot**
    
- terminal sockets remain **enabled**
    
- users can connect via **`m5term`**
    

---

# Summary of Script Modifications

Three modifications were applied to the original `run_spec.py` script:

1. **Disable benchmark script injection**  → [[# 1. Benchmark Script Injection Disabled]] 
    
2. **Prevent listener sockets from being disabled**  → [[# 2. Listener Disabling Removed]]
    
3. **Add an infinite simulation loop after boot** → [[#3. Infinite Simulation Loop Added]]
    

These changes convert the script from **automatic SPEC execution** to **interactive debugging mode**.

---

# 1. Benchmark Script Injection Disabled

### Original Code

```python
system.readfile = writeBenchScript(
    m5.options.outdir,
    benchmark_name,
    benchmark_size,
    output_dir
)
```

### Modified Code (Comment it out)

```python
# system.readfile = writeBenchScript(
#     m5.options.outdir,
#     benchmark_name,
#     benchmark_size,
#     output_dir
# )
```

### What This Code Does

In the original script:

1. A benchmark script is generated, such as:
    

```
run_500.perlbench_r_test
```

2. This script is passed to the guest OS using:
    

```
system.readfile
```

3. The guest system's **rcS startup script reads this file and executes the benchmark automatically**.
    

### Effect of the Modification

By commenting out this line:

- No SPEC script is injected into the guest system
- Linux boots normally
- **No benchmark is executed automatically** 

This is the **primary change that disables SPEC execution**.

---

# 2. Listener Disabling Removed

### Original Code

```python
if not allow_listeners:
    m5.disableAllListeners()
```

### Modified Code

```python
# if not allow_listeners:
#     m5.disableAllListeners()
```

### What This Code Does

Originally:

1. gem5 enables sockets based on CLI options
2. The Python script disables them using:

```
m5.disableAllListeners()
```

Result:

```
Sockets disabled, not accepting terminal connections
```

### Effect of the Modification

By commenting out this block:

- gem5 sockets remain enabled
- the terminal interface stays available

This allows external tools such as:

```
m5term
```

to connect to the simulation.

---

# 3. Infinite Simulation Loop Added

### New Code Added

```python
print("Interactive FS boot complete.")
while True:
    m5.simulate()
```

### Purpose

This loop keeps the gem5 simulation alive indefinitely.

### Original Script Flow

```
Boot Linux
↓
Inject SPEC script
↓
Run benchmark
↓
Dump statistics
↓
Copy logs
↓
Terminate simulation
```

### Modified Script Flow

```
Boot Linux
↓
Do NOT inject SPEC script
↓
Do NOT disable listeners
↓
Enter infinite simulation loop
↓
Allow terminal interaction
```

---

# Why the Infinite Loop Works

The call:

```python
m5.simulate()
```

does **not spin the CPU continuously**.

Instead, gem5 advances simulation **until the next event occurs**.

If no events occur, gem5 simply waits.

Therefore the loop effectively behaves like:

```
wait forever for guest activity
```

This is a **common technique used for interactive gem5 debugging setups**.

---

# SPEC Execution Is Now Completely Disabled

Because of the changes above, SPEC execution cannot start.

Three safeguards now exist:

### 1. Benchmark Script Injection Removed

```
system.readfile
```

is disabled.

---

### 2. Benchmark Execution Code Is Never Reached

After boot, the script enters:

```python
while True:
    m5.simulate()
```

The remaining benchmark logic is never executed.

---

### 3. CPU Switching and Benchmark Launch Are Skipped

These functions become unreachable:

```
switchCpus()
run_spec_benchmark()
copy_spec_logs()
```

---

# Resulting Interactive Workflow

After the modifications, the system behaves as follows:

```
Host Machine
     │
     ▼
Start gem5
     │
     ▼
Linux guest boots
     │
     ▼
"Interactive FS boot complete"
     │
     ▼
gem5 waits for interaction
     │
     ▼
User connects using m5term
```

---

# Connecting to the Guest System

Now in VS code run first activate the virtual env:
```
 source gem5_py38_env/bin/activate
```

Then start the GEM5 FS simulation:
```
 ./run_fs_Debug.sh 
```

Once gem5 prints:
```
Interactive FS boot complete.
```


![[Pasted image 20260316005438.png]]


connect to the guest console using:

```bash
~/Desktop/spec_project/spec_2017/gem5/util/term/m5term localhost 3456
```

Expected shell:
![[Pasted image 20260316005516.png]]

```
root@gem5-host:/#
```

At this point:

- SPEC is **not running**
- gem5 is **fully interactive**
- kernel modules can be compiled
- system debugging is possible

---

# Why This Modification Was Necessary

Without modifying the script:

|Behavior|Result|
|---|---|
|SPEC auto-executes|No interactive shell|
|sockets disabled|`m5term` cannot connect|
|simulation exits after benchmark|debugging impossible|


---

# Related Documentation

- [[gem5_setup]]
    
- [[memguard_setup]]
    
- [[interactive_terminal_gem5]]
    

---
