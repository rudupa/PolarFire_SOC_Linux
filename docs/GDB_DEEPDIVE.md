# GDB Deep Dive for Embedded Linux and Autonomous Vehicle Platforms

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. What GDB Solves](#2-what-gdb-solves)
- [3. Configuration Levels: Partial vs Full](#3-configuration-levels-partial-vs-full)
- [4. Build and Symbol Strategy](#4-build-and-symbol-strategy)
- [5. Target Setup Patterns](#5-target-setup-patterns)
- [6. Core Usage Workflows](#6-core-usage-workflows)
- [7. AV-Centric Use Cases](#7-av-centric-use-cases)
- [8. GDB with systemd Services](#8-gdb-with-systemd-services)
- [9. Remote Debugging with gdbserver](#9-remote-debugging-with-gdbserver)
- [10. Post-Mortem Debugging (Core Dumps)](#10-post-mortem-debugging-core-dumps)
- [11. Multi-Thread and Concurrency Debugging](#11-multi-thread-and-concurrency-debugging)
- [12. Best Practices](#12-best-practices)
- [13. Common Pitfalls and Fixes](#13-common-pitfalls-and-fixes)
- [14. Handy Command Reference](#14-handy-command-reference)

## 1. Purpose

This guide focuses on practical GDB usage in embedded Linux systems, especially for Autonomous Vehicle (AV) software stacks where deterministic behavior, reliability, and traceability are critical.

## 2. What GDB Solves

GDB helps with:
- Crash root-cause analysis
- Logic errors and bad state transitions
- Thread synchronization issues
- Memory corruption detection (with supporting tools)
- Remote target debugging when direct host execution is not possible

In AV systems, this is essential for perception, planning, control, gateway, and middleware components.

## 3. Configuration Levels: Partial vs Full

### 3.1 Partial configuration

Use partial configuration when you need low overhead and targeted debugging.

Typical setup:
- Build binaries with frame pointers and minimal symbols for selected components
- Enable gdbserver only on development images
- Keep optimization enabled for most packages

Recommended for:
- On-vehicle quick triage
- Reproducing a specific bug with minimal system impact

### 3.2 Full configuration

Use full configuration for deep investigations and hard bugs.

Typical setup:
- Full debug symbols for application and middleware components
- Core dump collection enabled
- Debug-friendly compiler flags for relevant modules
- Optional sanitizers in non-production builds

Recommended for:
- Complex timing bugs
- Rare race conditions
- Intermittent crashes that require post-mortem analysis

### 3.3 Decision rule

Start with partial configuration for fast iteration, then move to full configuration if you cannot isolate the issue within one to two debugging cycles.

## 4. Build and Symbol Strategy

### 4.1 Keep symbolized artifacts

For each release or candidate build, retain:
- Exact executable files
- Debug symbols
- Build ID and toolchain metadata
- Matching source revision

### 4.2 Suggested build profiles

- dev-debug: low optimization, rich symbols, assert enabled
- qa-debug: near-production optimization, selected symbols
- prod: optimized, stripped runtime binaries, external symbol package retained

### 4.3 Compiler flag guidance

For debuggability of critical modules, prefer:
- -g or -g3 for symbols
- -fno-omit-frame-pointer for easier stack analysis
- Controlled optimization levels for stable variable inspection

## 5. Target Setup Patterns

### 5.1 Local process debugging

Use when the binary can run directly in your dev environment.

### 5.2 Remote target debugging

Use when the software runs on embedded target hardware.

Pattern:
1. Launch target program under gdbserver on device
2. Connect from host gdb with matching binary and symbols
3. Set breakpoints and inspect state remotely

### 5.3 Attach to running process

Best for long-running services where restart affects behavior.

## 6. Core Usage Workflows

### 6.1 Crash reproduction workflow

1. Reproduce with deterministic input
2. Break on signals or exceptions
3. Capture full backtrace with local variables
4. Inspect calling path and shared state
5. Validate fix by re-running same scenario

### 6.2 Regression verification workflow

1. Keep a known failing input
2. Apply fix
3. Re-run under gdb with key watchpoints
4. Confirm control path now follows expected state transitions

## 7. AV-Centric Use Cases

### 7.1 Sensor ingest crashes

Symptoms:
- Process crash after malformed frame or timing spike

GDB focus:
- Buffer bounds
- Null and stale pointers
- Thread ownership of frame buffers

### 7.2 Planning node deadlocks

Symptoms:
- Node appears alive but output stalls

GDB focus:
- Thread backtraces for all threads
- Mutex ownership and wait chains
- Condition variable signaling paths

### 7.3 Control loop timing anomalies

Symptoms:
- Missed deadlines under high load

GDB focus:
- Blocking calls in critical path
- Unexpected allocations or logging in real-time loop
- Priority inversion indicators with supporting system tools

### 7.4 Middleware bridge faults

Symptoms:
- DDS to MQTT or DDS to Zenoh bridge drops data

GDB focus:
- Serialization and deserialization paths
- Queue growth and drop logic
- Reconnect and retry state machines

## 8. GDB with systemd Services

### 8.1 Debug restart-looping services

Recommended approach:
- Temporarily disable aggressive restart policy
- Run the same command line manually under gdb
- Re-enable service policy after diagnosis

### 8.2 Debug in service context

If behavior depends on service environment, copy relevant environment values and runtime paths into your debug session.

## 9. Remote Debugging with gdbserver

Typical model:
- Target: gdbserver :2345 /path/to/app arg1 arg2
- Host: gdb /path/to/app
- Host connect command: target remote <target-ip>:2345

Tips:
- Ensure host and target binaries match exactly
- Keep sysroot and shared libraries aligned
- Use a dedicated debug network if possible

## 10. Post-Mortem Debugging (Core Dumps)

### 10.1 Why post-mortem matters

Some production failures cannot be reproduced interactively. Core dump analysis provides the exact crashed state.

### 10.2 Minimal process

1. Collect core dump and binary build ID
2. Load binary plus core into gdb
3. Extract backtrace for all threads
4. Inspect key locals and global state
5. Correlate with logs and telemetry timeline

## 11. Multi-Thread and Concurrency Debugging

Useful tactics:
- Inspect all threads first before deep diving one thread
- Identify lock order and waiting dependencies
- Use conditional breakpoints to reduce noise
- Watch shared counters and queue pointers carefully

For race conditions, combine gdb with sanitizers and trace tools in debug builds.

## 12. Best Practices

- Keep symbols for every shipped build
- Standardize incident collection: logs, core, binary hash, config
- Avoid debugging with mismatched binaries and symbols
- Prefer reproducible scenarios over ad-hoc manual poking
- Use scripted gdb command files for repeatable triage
- Separate debug and production images clearly
- Limit debugging changes in real-time control paths
- Document each root cause and add regression tests

## 13. Common Pitfalls and Fixes

- Pitfall: Variables show as optimized out
  - Fix: use lower optimization for target modules in debug profile

- Pitfall: Backtrace is incomplete
  - Fix: enable frame pointers and verify symbol availability

- Pitfall: Breakpoints not hit on target
  - Fix: verify binary path, loaded symbols, and process instance

- Pitfall: Different behavior under debugger
  - Fix: use post-mortem and trace-based methods to validate findings

## 14. Handy Command Reference

Useful commands to remember:
- run, continue, next, step, finish
- break <location>, delete, disable, enable
- bt, bt full, info threads, thread apply all bt
- print <expr>, display <expr>, watch <expr>
- info locals, info args, info registers
- set pagination off
- set logging on

Remote and post-mortem essentials:
- target remote <ip:port>
- file <binary>
- core <corefile>

For AV programs, keep a short internal playbook that maps these commands to your most common failure classes: crash, deadlock, timing drift, and data corruption.
