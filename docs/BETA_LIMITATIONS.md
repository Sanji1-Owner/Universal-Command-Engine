# UCE v0.1 Beta Limitations

This document outlines the known limitations and constraints of UCE v0.1 Beta.

## Feature Limitations

### Platform Support

**Currently Supported:**
- ✅ Windows 7 SP1 and later
- ✅ 64-bit (x64) architecture (x86 available but untested)
- ✅ Python 3.9, 3.10, 3.11

**Not Supported (v0.1):**
- ❌ macOS (planned for v1.0)
- ❌ Linux (planned for v1.0)
- ❌ Python 3.8 and earlier
- ❌ Python 3.12+ (untested)
- ❌ Alternative Python implementations (PyPy, etc.)

### Execution Modes

**Supported:**
- ✅ Python script execution
- ✅ Command execution (basic)
- ✅ Local isolated execution

**Not Supported (v0.1):**
- ❌ Remote execution (planned for v1.0)
- ❌ Distributed execution (v2.0+)
- ❌ JavaScript/Node.js (v1.0+)
- ❌ C#, Go, Rust backends (v1.0+)
- ❌ Interactive REPL mode
- ❌ Streaming output (capture entire output only)

### Output and Communication

**Supported:**
- ✅ Complete stdout/stderr capture
- ✅ Standard Python print() statements
- ✅ Structured result delivery

**Not Supported (v0.1):**
- ❌ Real-time output streaming
- ❌ Binary data in output (text only)
- ❌ Output larger than 10 MB per execution
- ❌ Interactive input/stdin

### API Features

**Supported:**
- ✅ Synchronous execution
- ✅ Memory and timeout limits
- ✅ File access restrictions
- ✅ Sandbox configuration

**Not Supported (v0.1):**
- ❌ Asynchronous execution (planned)
- ❌ Concurrent execution from same engine
- ❌ Execution history/logging API
- ❌ Plugin system (v1.0+)
- ❌ Custom sandbox rules

## Performance Limitations

### Process Creation Overhead

- **First execution**: 50-200 ms (includes Python startup)
- **No process pooling**: Each execution creates new process
- **No caching**: No intermediate state preservation

**Implication:** Not suitable for ultra-high-frequency execution (>100 scripts/sec)

### Memory Overhead

- **Per-engine overhead**: ~5-10 MB
- **Per-child overhead**: Configured minimum (default 8 MB)
- **No memory pooling**: Each execution has full overhead

**Implication:** Not suitable for extremely memory-constrained environments

### Output Size Limits

- **Maximum per execution**: 10 MB stdout + 10 MB stderr
- **No streaming**: Must capture entire output in memory
- **No compression**: Output stored uncompressed

**Implication:** Not suitable for very large output scripts

### Execution Limits

- **Timeout precision**: ±50 ms (OS dependent, not real-time)
- **No CPU limits**: CPU usage not constrained (Job Objects capable but not exposed)
- **No disk I/O limits**: Disk usage not constrained
- **No network limits**: Network access not constrained (blocked via sandbox)

## Security Limitations

### Sandbox Design

⚠️ **IMPORTANT**: The sandbox prevents **accidental** misuse, not **determined** attacks.

**Does protect against:**
- ✅ Accidental infinite loops (timeout)
- ✅ Accidental memory exhaustion (memory limit)
- ✅ Accidental file system access (path restrictions)
- ✅ Accidental process spawning (subprocess block)
- ✅ Accidental system API misuse (os/sys block)

**Does NOT protect against:**
- ❌ Exploits of Python interpreter itself
- ❌ Windows/OS kernel vulnerabilities
- ❌ Side-channel attacks
- ❌ Determined attackers with system knowledge
- ❌ Compromised host operating system

### Threat Model Scope

UCE is suitable for:
- Scripts written in good faith (cooperative)
- Preventing accidental errors and resource exhaustion
- Integrated execution within trusted applications

UCE is NOT suitable for:
- Adversarial/malicious scripts
- Scripts from untrusted external sources
- Security-critical isolation (use containers/VMs instead)
- Untrusted code execution in production

### Known Sandbox Restrictions

The following are blocked for safety:

```python
# Blocked
import os              # No filesystem/process access
import sys             # No system info access
import subprocess      # No process spawning
exec("code")          # No dynamic code execution
eval("expr")          # No dynamic evaluation
getattr(obj, "attr")  # No attribute tricks
__import__('os')      # No dynamic imports
```

Users must work within these constraints. Scripts cannot be modified to bypass sandbox at runtime.

## Configuration Limitations

### Memory Configuration

- **Minimum limit**: 8 MB (child process needs to start)
- **Maximum limit**: System RAM (typically 256 GB on modern systems)
- **Granularity**: 1 byte (any value allowed)
- **No per-script tuning**: Limit applies to entire script

**Implication:** Cannot set very low limits; scripts need minimum viable memory

### Timeout Configuration

- **Minimum timeout**: 100 ms (practical minimum)
- **Maximum timeout**: 24 hours (231 - 1 milliseconds)
- **Granularity**: 1 millisecond
- **Precision**: ±50 ms (OS dependent)

**Implication:** Timeouts are approximate, not precise; system load affects precision

### File Access Configuration

- **Restriction granularity**: Directory path level
- **No file-level access control**: Can't restrict specific files
- **No permission levels**: Whitelisted directory = full read/write access
- **No time-based expiry**: Access whitelist doesn't expire

**Implication:** File restrictions are coarse-grained, not file-level

## API Limitations

### Thread Safety

- **Not thread-safe**: Single-threaded use required
- **Solution**: Create separate `CommandEngine` instance per thread
- **No mutex**: Caller responsible for synchronization

**Implication:** Cannot share engine instance across threads

### State Management

- **No execution history**: Results not cached
- **No script reuse**: Each execution is independent
- **No state persistence**: No inter-execution state

**Implication:** Cannot access execution history or maintain state between executions

### Error Handling

- **Limited error detail**: Error messages are string-based
- **No error codes**: Only status enum (no numeric codes)
- **No error recovery**: Caller must decide retry strategy

**Implication:** Error handling is basic; advanced diagnostics not available

## Deployment Limitations

### Binary Dependencies

- **Windows-only binaries**: Not redistributable to non-Windows platforms
- **Runtime dependencies**: Python 3.9+ must be installed separately
- **DLL versioning**: All DLLs must be from same v0.1 release

**Implication:** Tight coupling to specific Windows versions and Python

### Installation Requirements

- **Manual setup**: No MSI installer (v0.1 beta)
- **PATH configuration**: User must configure environment
- **No automatic updates**: Manual version upgrades needed

**Implication:** Not suitable for end-user deployment without scripts

## Known Issues

### Windows-Specific

- **Job Object compatibility**: Requires Windows 7 SP1+
- **UAC interaction**: Some operations may require admin rights
- **Terminal compatibility**: Not tested with all terminal emulators

### Python-Specific

- **Import performance**: First import slower than subsequent (caching not visible)
- **Memory reporting**: Inaccuracy in process memory tracking (OS variance)
- **Startup time variance**: Depends on system load and disk performance

### IPC-Specific

- **Message size limits**: Large outputs may be truncated (10 MB limit)
- **Error message truncation**: Long errors truncated in output
- **Timeout precision**: System load affects timeout accuracy

## What's NOT Available (v0.1)

### Missing v1.0 Features

❌ Cross-platform support (Mac/Linux)
❌ Plugin architecture
❌ Asynchronous execution
❌ JavaScript/Node.js backend
❌ Advanced monitoring API
❌ Commercial support
❌ Third-party audit (pending)

### Missing Advanced Features (v2.0+)

❌ Distributed execution
❌ Container integration
❌ Custom sandbox policies
❌ Performance optimizations (25%+ throughput)
❌ Cloud execution backend

## Workarounds and Mitigations

### For High Frequency Execution

- **Issue**: Process creation overhead too high
- **Workaround**: Batch scripts together into single execution
- **Future**: Process pooling in v1.0+

### For Large Output

- **Issue**: 10 MB output limit
- **Workaround**: Write output to file instead of stdout
- **Future**: Streaming output in v1.0+

### For Interactive Execution

- **Issue**: No stdin/interactive support
- **Workaround**: Pre-generate all input as script variables
- **Future**: Possible in v1.0+ with architectural changes

### For Multi-Platform

- **Issue**: Windows only
- **Workaround**: Use separate deployment for each platform
- **Future**: macOS and Linux support in v1.0

## Recommendations

### For Production Use

- ✅ Do: Use for cooperative scripts in trusted applications
- ✅ Do: Set appropriate memory/timeout limits
- ✅ Do: Monitor and log execution results
- ✅ Do: Use for features, not security

- ❌ Don't: Rely on sandbox for adversarial code isolation
- ❌ Don't: Execute untrusted scripts without review
- ❌ Don't: Assume security boundaries at script level
- ❌ Don't: Deploy without fallback mechanisms

### For Evaluation

- ✅ Do: Test with your typical scripts
- ✅ Do: Monitor memory and timing
- ✅ Do: Review sandbox restrictions
- ✅ Do: Provide feedback on limitations

- ❌ Don't: Assume v0.1 stability guarantees
- ❌ Don't: Deploy to production without thorough testing
- ❌ Don't: Make long-term commitments on beta API

## Timeline for Limitation Resolution

### v1.0 (June 24, 2026)

- macOS and Linux support
- Async execution API
- Plugin system foundation
- JavaScript/Node.js backend (preview)

### v1.x (2026-2027)

- Performance optimizations (25%+ improvement)
- Streaming output support
- Advanced monitoring API
- Commercial licensing

### v2.0+ (2027+)

- Distributed execution
- Container integration
- Custom sandbox policies
- Cloud execution

---

**UCE v0.1 Beta** is a pre-release version with known limitations. Use for evaluation and integration testing; full production deployment planned for v1.0 (June 24, 2026).

For security limitations, see [../SECURITY.md](../SECURITY.md).
For roadmap, see [../ROADMAP.md](../ROADMAP.md).
