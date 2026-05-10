# UCE Frequently Asked Questions

## Installation and Setup

### Q: Where do I download UCE v0.1 Beta?

**A:** Download from GitHub releases: `github.com/Sanji1-Owner/Universal-Command-Engine/releases`

Look for the `uce-v0.1-beta-windows-x64.zip` file.

See [INSTALLATION.md](../INSTALLATION.md) for detailed setup instructions.

### Q: What are the system requirements?

**A:** Minimum:
- Windows 7 SP1 or later
- 2 GB RAM
- 50 MB disk space
- Python 3.9 or later (for Python scripts)

**Recommended:**
- Windows 10 or later
- 4 GB RAM
- 100 MB disk space
- Python 3.10 or 3.11

### Q: Do I need to compile from source?

**A:** No. UCE v0.1 Beta is distributed as pre-built binaries:
- `uce_command_engine.dll` (compiled binary)
- `uce_script_runner.exe` (compiled binary)
- `uce_command_engine.lib` (import library for linking)

Just extract and link against the library.

### Q: Can I use UCE on macOS or Linux?

**A:** Not in v0.1 Beta. Windows only.

macOS and Linux support is planned for v1.0 (June 24, 2026).

### Q: How do I add UCE to my project?

**A:** See [EMBEDDING_GUIDE.md](../EMBEDDING_GUIDE.md) for detailed integration steps.

Quick summary:
1. Download and extract UCE
2. Add include path: `include/uce/`
3. Link against: `uce_command_engine.lib`
4. Copy DLLs to build output directory

## Usage and API

### Q: How do I execute a Python script?

**A:**

```cpp
#include <uce/CommandEngine.h>

uce::CommandEngine engine;
engine.SetMemoryLimit(256 * 1024 * 1024);
engine.SetTimeout(5000);

uce::ExecutionResult result = engine.ExecutePython(
    "print('Hello from UCE')"
);

if (result.status == uce::ExecutionStatus::Success) {
    std::cout << result.stdout_output << std::endl;
}
```

See [examples/basic_command.md](../examples/basic_command.md) for more.

### Q: How do I set memory limits?

**A:**

```cpp
uce::CommandEngine engine;

// 256 MB limit
engine.SetMemoryLimit(256 * 1024 * 1024);

// No limit (default)
engine.SetMemoryLimit(0);

// Minimum 8 MB (child process minimum)
engine.SetMemoryLimit(8 * 1024 * 1024);
```

See [examples/memory_limit.md](../examples/memory_limit.md) for examples.

### Q: How do I set execution timeouts?

**A:**

```cpp
uce::CommandEngine engine;

// 5 second timeout
engine.SetTimeout(5000);

// 30 second timeout
engine.SetTimeout(30000);

// No timeout (default)
engine.SetTimeout(0);

// Minimum 100 ms practical
engine.SetTimeout(100);
```

### Q: Can I execute multiple scripts concurrently?

**A:** No. Create separate `CommandEngine` instances for each script:

```cpp
// Safe: Each thread has its own instance
std::thread t1([]() { 
    uce::CommandEngine engine;
    engine.ExecutePython("...");
});
std::thread t2([]() {
    uce::CommandEngine engine;
    engine.ExecutePython("...");
});
```

### Q: How do I handle errors?

**A:**

```cpp
uce::ExecutionResult result = engine.ExecutePython(script);

switch (result.status) {
    case uce::ExecutionStatus::Success:
        std::cout << result.stdout_output << std::endl;
        break;
    case uce::ExecutionStatus::Timeout:
        std::cerr << "Script timed out" << std::endl;
        break;
    case uce::ExecutionStatus::MemoryLimitExceeded:
        std::cerr << "Memory limit exceeded" << std::endl;
        break;
    case uce::ExecutionStatus::SandboxViolation:
        std::cerr << "Sandbox violation: " << result.error_message << std::endl;
        break;
    default:
        std::cerr << "Error: " << result.error_message << std::endl;
}
```

## Sandbox and Security

### Q: What Python operations are blocked?

**A:** Blocked imports:
- `import os` — No OS access
- `import sys` — No system info
- `import subprocess` — No process spawning
- `import importlib` — No dynamic imports

Blocked functions:
- `exec()` — No dynamic code execution
- `eval()` — No expression evaluation
- `getattr()` — No attribute tricks (for security)
- `__import__()` — No dynamic imports

Everything else is allowed (math, string, list, dict, json, etc.).

See [examples/sandbox_violation.md](../examples/sandbox_violation.md) for demo.

### Q: Is UCE secure for untrusted scripts?

**A:** No. The sandbox prevents **accidental** errors, not **determined** attacks.

✅ Use UCE for: Cooperative scripts you write or trust
❌ Don't use UCE for: Adversarial scripts from untrusted sources

For adversarial code, use OS-level isolation (containers, VMs).

See [SECURITY.md](../SECURITY.md) for security details.

### Q: Can my script access the filesystem?

**A:** By default, file access is blocked (os module blocked).

But you can read data passed into the script or write to specific configured paths.

For file operations, consider:
1. Pass data as script variables
2. Write output to specific files
3. Host application manages file I/O

### Q: Can my script call external programs?

**A:** No. `subprocess` module is blocked.

Script can only use Python libraries and functions.

### Q: What if I need a blocked operation?

**A:** Blocked operations are intentional for safety.

Options:
1. **Modify script**: Work within sandbox restrictions
2. **Integrate into host**: Perform OS operations in host application
3. **Contribute feedback**: Report limitations for future versions
4. **Wait for v1.0**: Enhanced capabilities may be available

## Performance

### Q: How slow is the process creation overhead?

**A:** Process creation typically takes **50-200 ms**:
- First execution: 200 ms (includes Python startup)
- Subsequent executions: 50-100 ms (depends on system load)

**Impact:** Fine for occasional script execution, not for high-frequency (>100/sec).

**Workaround**: Batch multiple scripts into single execution.

### Q: How much memory overhead does UCE add?

**A:** Per execution:
- Parent overhead: ~5-10 MB (engine structures)
- Child overhead: ~8-20 MB (Python startup + script data)
- Total: ~15-30 MB per execution

**Impact:** Minimal for typical applications.

### Q: What's the IPC latency?

**A:** Inter-process communication latency is typically **<5 ms**.

Not a bottleneck for most use cases.

### Q: How much does output capture slow things down?

**A:** Output capture is **<1% overhead** for typical scripts.

Maximum output: 10 MB per execution (hard limit).

## Troubleshooting

### Q: I get "uce_command_engine.dll not found"

**A:** The DLL is not in your application directory or PATH.

Solutions:
1. Copy DLL to same directory as your .exe
2. Add DLL directory to system PATH
3. Verify DLL filename matches exactly

### Q: My script gets "SandboxViolation" error

**A:** Your script tried a blocked operation (os, sys, subprocess, exec, eval, etc.).

Solutions:
1. Remove `import os` or other blocked imports
2. Modify script to work within sandbox
3. Perform OS operations in host application instead
4. See [examples/sandbox_violation.md](../examples/sandbox_violation.md)

### Q: My script times out but should be fast

**A:** Timeout might be too short, or system might be slow.

Solutions:
1. Increase timeout: `engine.SetTimeout(10000)` (10 seconds)
2. Optimize script for performance
3. Check system resource usage (CPU, disk I/O)

### Q: My script hits memory limit but should fit

**A:** Python has memory overhead; scripts need ~10-20 MB just to start.

Solutions:
1. Increase memory limit: `engine.SetMemoryLimit(512 * 1024 * 1024)`
2. Optimize script memory usage
3. Break large data processing into smaller batches

### Q: IPC error when executing

**A:** Likely mismatch between DLL versions or process creation failure.

Solutions:
1. Verify all DLLs are from same v0.1 release
2. Check Windows UAC (User Account Control) isn't blocking
3. Ensure `uce_script_runner.exe` exists in `bin/` directory
4. Try running as administrator

## Beta and Versioning

### Q: Is v0.1 Beta production-ready?

**A:** No. This is a pre-release version for evaluation and testing.

- Not recommended for mission-critical production
- API may change before v1.0
- Third-party security audit pending
- Use with fallback mechanisms

### Q: When will v1.0 be released?

**A:** **June 24, 2026**

See [ROADMAP.md](../ROADMAP.md) for details.

### Q: What's the difference between v0.1 Beta and v1.0?

**A:** v1.0 planned additions:
- Windows, macOS, Linux support
- Async execution API
- Plugin architecture
- JavaScript/Node.js backend
- Third-party security audit
- Production-grade stability

v0.1 is evaluation-focused; v1.0 targets production use.

### Q: Can I use v0.1 in production?

**A:** At your own risk. Consider:
- ✅ Use with fallback mechanisms
- ✅ Thoroughly test your scripts
- ✅ Monitor execution in production
- ✅ Plan upgrade path to v1.0
- ❌ Don't rely on beta guarantees
- ❌ Don't use for mission-critical systems

### Q: Will my v0.1 code work with v1.0?

**A:** Likely, but not guaranteed. Public API may change before v1.0.

Monitor releases for migration guides.

## Support and Contribution

### Q: How do I report a bug?

**A:** File an issue on GitHub: `github.com/Sanji1-Owner/Universal-Command-Engine/issues`

Include:
- Steps to reproduce
- Expected vs. actual behavior
- System info (Windows version, Python version)
- Code snippet if applicable

### Q: How do I report a security issue?

**A:** Do NOT create public issues for security vulnerabilities.

See [SECURITY.md](../SECURITY.md#reporting-security-issues) for reporting process.

### Q: How can I contribute?

**A:** UCE v0.1 Beta is a closed-source pre-release.

Contributions planned for open-source roadmap (post-v1.0).

For now, provide feedback via GitHub issues.

### Q: Where can I get more help?

**A:**
- Check [examples/](../examples/) for code samples
- Read [EMBEDDING_GUIDE.md](../EMBEDDING_GUIDE.md) for integration
- Review [API.md](../API.md) for detailed API reference
- See [SECURITY.md](../SECURITY.md) for security questions

### Q: Is there commercial support?

**A:** Not in v0.1 Beta. Planned for v1.0+ releases.

---

**More questions?** File an issue: `github.com/Sanji1-Owner/Universal-Command-Engine/issues`

For security questions, see [SECURITY.md](../SECURITY.md).
