# UCE v0.1 Beta Release Notes

## Release Overview

**Universal Command Engine (UCE) v0.1 Beta** is the first public release of the closed-source SDK for isolated command and script execution on Windows.

**Release Date**: May 9, 2026  
**Status**: Beta (pre-production)  
**Target Stability**: v1.0 (June 24, 2026)

## What's New

### Core Features

- **Isolated execution**: Run commands and scripts in separate child processes
- **Python support**: Execute Python 3.9+ scripts with sandbox protections
- **Memory enforcement**: OS-level memory limits prevent exhaustion attacks
- **Timeout protection**: Configurable execution timeouts
- **Sandbox model**: Block dangerous imports and operations
- **Closed-source SDK**: Binary distribution with public headers only

### Verified in This Release

**Security:**
- IPC is protected and validated
- Sandbox import restrictions work
- File path traversal is prevented
- Memory limits are enforced
- Process cleanup completes properly

**Functionality:**
- 27 tests passed
- 0 tests failed
- 0 security violations detected

**Reliability:**
- No orphan processes
- Proper resource cleanup
- Graceful error handling

## System Requirements

- Windows 7 SP1 or later
- 2 GB RAM minimum (4 GB recommended)
- Python 3.9 or later (for Python script execution)
- 50 MB disk space

## Installation

1. Download `uce-v0.1-beta-windows-x64.zip` from releases
2. Extract to desired location (e.g., `C:\opt\uce`)
3. Add to PATH or reference in your project
4. See [INSTALLATION.md](INSTALLATION.md) for detailed steps

## Quick Start

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

For more examples, see [examples/](examples/) and [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md).

## Known Limitations

### Platform

- **Windows only** — macOS and Linux support planned for v1.0
- **Local execution only** — No remote code execution
- **Python only** — No C#, Go, Rust, or JavaScript backends yet

### Sandbox

- **Not for adversarial code** — Use OS-level isolation (containers/VMs) for untrusted code
- **Defense-in-depth** — Prevents accidental misuse, not determined attackers
- **Beta status** — Security audit pending for v1.0

### Features

- **No streaming output** — Captures entire output after execution
- **No async API** — Synchronous execution only
- **No plugins** — Plugin system planned for v1.0
- **No remote** — Local execution only

See [SECURITY.md](SECURITY.md#known-limitations) for security-specific limitations.

## Beta Status Warning

**This is a pre-release beta version.**

- **Not for production**: Use for evaluation and integration testing only
- **API subject to change**: Before v1.0 (June 24, 2026)
- **Security audit pending**: Third-party review planned for v1.0
- **Limited support**: Community support only, no SLAs

### Do NOT use for:
- Execution of untrusted/adversarial code in production
- Critical security-sensitive operations
- Production systems without fallback
- Systems requiring guaranteed uptime

### Do use for:
- Evaluation and testing
- Integration into applications
- Development and prototyping
- Feedback on functionality
- Security assessment

## Verified Test Results

All 27 tests passed with 0 failures:

```
Security Tests:        PASSED
  - Sandbox import restrictions
  - File path traversal prevention
  - Memory limit enforcement
  - Timeout enforcement
  - IPC message validation

Functionality Tests:    PASSED
  - Python script execution
  - Command execution
  - Output capture
  - Error handling
  - Process cleanup

Resource Tests:        PASSED
  - Memory usage tracking
  - Process termination
  - Child process cleanup
  - Orphan process prevention
```

See [CHANGELOG.md](CHANGELOG.md) for detailed verification status.

## Documentation

- **[README.md](README.md)** — Overview and features
- **[API.md](API.md)** — Complete API reference
- **[EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md)** — Integration guide
- **[INSTALLATION.md](INSTALLATION.md)** — Setup instructions
- **[SECURITY.md](SECURITY.md)** — Security model and disclosure
- **[ROADMAP.md](ROADMAP.md)** — Future versions
- **[CHANGELOG.md](CHANGELOG.md)** — Change history
- **[examples/](examples/)** — Code examples
- **[docs/](docs/)** — Additional documentation

## Support

### Getting Help

- See [docs/FAQ.md](docs/FAQ.md) for common questions
- Check [examples/](examples/) for code samples
- Review [API.md](API.md) for usage details

### Reporting Issues

For bugs or feature requests: github.com/Sanji1-Owner/Universal-Command-Engine/issues

For security issues: See [SECURITY.md](SECURITY.md#reporting-security-issues)

## License

UCE v0.1 Beta is distributed under a closed-source proprietary license.

See [PROPRIETARY_LICENSE.md](PROPRIETARY_LICENSE.md) for full licensing terms.

## Roadmap

### v0.2 (Q3 2026)
- Extended execution modes
- Improved diagnostics
- Performance tuning

### v0.5 (Q4 2026)
- macOS and Linux validation
- Advanced monitoring

### v1.0 (June 24, 2026)
- Production-ready stability
- Full cross-platform support
- Plugin ecosystem
- JavaScript/Node.js backend
- Third-party security audit
- Commercial licensing

See [ROADMAP.md](ROADMAP.md) for details.

## Migration Guide

No previous versions. This is the first release.

For future upgrades, see migration guides in respective version release notes.

## Feedback

We welcome feedback on UCE v0.1 Beta:

- Feature requests
- Integration experiences
- Performance observations
- Security findings

File GitHub issues or contact via release notes.

---

**Thank you for testing UCE v0.1 Beta!**

Questions? See [docs/FAQ.md](docs/FAQ.md)  
Next: UCE v0.2 (Q3 2026)  
Target: UCE v1.0 production (June 24, 2026)
