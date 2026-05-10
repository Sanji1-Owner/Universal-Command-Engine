# UCE Architecture Overview

This document provides a high-level overview of UCE v0.1 Beta architecture without exposing internal implementation details.

## Architecture Goals

- **Isolation**: Execute untrusted scripts in separate child processes
- **Enforcement**: Apply resource limits at OS level (not application level)
- **Security**: Sandbox dangerous operations while allowing safe Python
- **Simplicity**: Provide clean C++ API for host applications

## Design Principles

### 1. Process-Based Isolation

Scripts execute in **separate child processes**, not in-process.

**Benefits:**
- True memory isolation (not GC-based)
- Crash isolation (child crash doesn't crash parent)
- Process-level resource limits
- OS kernel enforces constraints

**Trade-offs:**
- Process creation overhead (50-200 ms)
- IPC communication for results
- Additional resource usage

### 2. OS-Level Enforcement

Resource limits are enforced **by the operating system**, not by UCE code.

**Memory enforcement** via Windows Job Objects:
- Set limit before child process starts
- OS kernel prevents exceeding limit
- Cannot be bypassed by application code

**Timeout enforcement** via UCE scheduler:
- Parent monitors child execution
- Terminates child if timeout exceeded
- Graceful shutdown with resource cleanup

### 3. Sandbox Model

Dangerous Python operations are **blocked before execution**, not monitored during.

**Blocked operations:**
- `import os` — filesystem and process access
- `import sys` — system introspection
- `import subprocess` — process spawning
- `exec()` and `eval()` — dynamic code execution

**Why proactive blocking:**
- Faster than monitoring
- Simpler to verify correctness
- Less chance of escape routes

### 4. Minimal Trust

The sandbox assumes **any script could be malicious** and validates accordingly.

However, UCE does NOT protect against:
- Python interpreter exploits
- OS kernel vulnerabilities
- Determined attackers with system access
- Side-channel attacks

## System Architecture

### High-Level Flow

```
Host Application
    ↓
[CommandEngine API]
    ↓
[Parent Process Control]
    ↓ (fork/exec)
[Child Process Sandbox]
    ↓
[Python Interpreter]
    ↓ (script execution)
[Output/IPC]
    ↓
[Result Delivery]
    ↓
Host Application
```

### Component Overview

#### 1. Public API Layer

- **CommandEngine** — Main execution interface
- **ScriptRunner** — Python-specific execution
- **ExecutionResult** — Execution results and metadata

What's exposed:
- ✅ Execution functions
- ✅ Resource configuration
- ✅ Result structures

What's NOT exposed:
- ❌ Internal IPC protocol
- ❌ Sandbox implementation details
- ❌ Memory management strategies

#### 2. Process Management Layer

Manages parent-child process lifecycle:

- Process creation with inherited constraints
- IPC channel setup before execution
- Metadata flow from parent to child
- Process termination and cleanup
- Resource monitoring

#### 3. Resource Enforcement Layer

Applies constraints at OS level:

- **Memory limits**: Windows Job Object memory enforcement
- **Timeout**: Parent-side scheduler with child termination
- **File access**: Restricted to configured paths
- **Process isolation**: Separate address space and resources

#### 4. Sandbox Layer

Prevents dangerous operations:

- Import blocking (os, sys, subprocess, importlib)
- Builtin restrictions (exec, eval, getattr)
- File path validation
- IPC message filtering

#### 5. Python Execution Layer

Manages Python interpreter within sandbox:

- Python 3.9+ startup and initialization
- Script code execution in sandbox context
- Output capture (stdout/stderr)
- Error handling and reporting

## Isolation Model

### Memory Isolation

```
Host Process (Parent)          Child Process (Script)
├─ .text (code)                ├─ .text (Python)
├─ .data (global data)         ├─ .data (runtime)
├─ Stack                       ├─ Stack
├─ Heap (Engine)               ├─ Heap (Script)
└─ IPC buffer                  └─ IPC channel
```

- **Separate address spaces**: No direct memory access
- **Job Object limit**: OS prevents exceeding allocated memory
- **IPC channel**: Structured communication with validation

### Execution Isolation

```
Parent Process                 Child Process
├─ Execution loop             ├─ Script startup
├─ Job Object setup           ├─ Sandbox initialization
├─ Child process fork ────→   ├─ Script execution
├─ IPC listen                 ├─ Result collection
├─ Timeout monitor            ├─ Output capture
├─ Result collection ←──── ├─ Result delivery
└─ Cleanup                     └─ Process exit
```

### Sandbox Isolation

Layers of sandbox protection:

1. **Import blocking** — Block dangerous modules at import time
2. **Builtin restrictions** — Prevent exec/eval/getattr
3. **File access** — Validate all file operations
4. **IPC filtering** — Validate all inter-process messages

## Resource Model

### Memory

- **Parent overhead**: ~5-10 MB (engine structures)
- **Child minimum**: 8 MB (Python startup)
- **Child maximum**: Configurable via SetMemoryLimit()
- **Limit enforcement**: Windows Job Object (OS kernel)

### Processes

- **Parent process**: Host application (1 per engine instance)
- **Child processes**: One per execution (created/destroyed per call)
- **Orphan prevention**: Automatic cleanup on engine destruction

### IPC

- **Channel type**: Windows named pipes or similar
- **Message format**: Structured with type/length headers
- **Capacity**: Designed for typical script output (<10 MB)
- **Validation**: Type checking and bounds validation

### Timing

- **Process creation**: 50-200 ms (includes Python startup)
- **Execution**: Variable (script-dependent)
- **Cleanup**: <10 ms (usually)
- **IPC latency**: <5 ms typical

## Security Design

### What We Protect Against

✅ Accidental script errors that crash host
✅ Unbounded memory allocations (via Job Objects)
✅ Infinite loops (via timeout)
✅ File system traversal (via path validation)
✅ Process spawning (via subprocess block)
✅ System access (via os/sys block)

### What We Do NOT Protect Against

❌ Python interpreter vulnerabilities
❌ OS kernel exploits
❌ Side-channel attacks (timing, etc.)
❌ Determined attackers with system knowledge
❌ Compromised host operating system
❌ Physical attacks on hardware

### Threat Model

UCE is designed for:
- **Cooperative scripts**: Scripts written in good faith
- **Accidental errors**: Preventing script bugs from crashing host
- **Integrated execution**: Scripts as part of application features

UCE is NOT designed for:
- **Adversarial code**: Intentionally malicious scripts
- **Untrusted sources**: Scripts from untrusted third parties
- **Security boundaries**: True sandboxing of untrusted code

For adversarial code, use OS-level isolation (containers, VMs, separate machines).

## Data Flow

### Execution Request

```
1. Host application creates ExecutionRequest
2. Passes to CommandEngine via ExecutePython()
3. Engine configures memory/timeout limits
4. Engine creates child process with constraints
5. Child receives script via IPC
```

### Execution Response

```
1. Child executes script in sandbox
2. Captures stdout/stderr
3. Monitors memory and timeout
4. Collects result metadata
5. Sends result via IPC
6. Parent receives and packages ExecutionResult
7. Returns to host application
```

### Error Handling

```
Success:     Script completes → stdout/stderr captured → Success status
Timeout:     Execution exceeds limit → child killed → Timeout status
Memory:      Exceeds Job Object limit → child killed → MemoryLimit status
Sandbox:     Blocked operation → error message → SandboxViolation status
Process:     Creation/management error → error message → ProcessError status
```

## Design Tradeoffs

### Chosen: Separate Processes

**Pros:** True isolation, OS enforcement, crash safety, security
**Cons:** Process overhead, IPC latency, resource usage

**Alternative:** In-process sandbox (not chosen)
- Pros: Lower overhead, no IPC latency
- Cons: No real isolation, GC-dependent, crash risk, harder to enforce limits

### Chosen: Proactive Sandbox

**Pros:** Fast, simple, verifiable, fewer escapes
**Cons:** More restrictive, user scripts might need modification

**Alternative:** Runtime monitoring (not chosen)
- Pros: Allows more operations
- Cons: Slower, complex, potential escapes, harder to verify

### Chosen: OS-Level Limits

**Pros:** Cannot be bypassed, enforced by kernel, reliable
**Cons:** Windows-specific (v0.1), not always granular

**Alternative:** Application-level tracking (not chosen)
- Pros: Cross-platform, flexible
- Cons: Can be bypassed, complex tracking, overhead

## Version Compatibility

### v0.1 Beta (Current)

- Windows only
- Process isolation
- Memory enforcement via Job Objects
- Python sandbox
- IPC-based communication

### v1.0 (Planned)

- Multi-platform (Windows, macOS, Linux)
- Plugin architecture
- Multiple execution backends
- Enhanced monitoring
- Advanced isolation options

### Future (v2.0+)

- Distributed execution
- Container integration
- Custom sandboxes
- Performance optimizations

---

**UCE v0.1 Beta Architecture**: Process-based isolation with OS-level enforcement and sandbox restrictions. Designed for cooperative scripts, not adversarial code.

For detailed API reference, see [API.md](../API.md).
For security details, see [SECURITY.md](../SECURITY.md).
