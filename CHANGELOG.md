# UCE Changelog

## v0.1 Beta (Current Release)

**Release Date**: May 9, 2026

### New Features

- **Core Command Engine**
  - Isolated child process execution
  - Command and Python script execution modes
  - Inter-process communication (IPC) for result delivery
  - Process metadata flow from parent to child

- **Memory Management**
  - OS-level memory limits via Windows Job Objects
  - Process memory monitoring and reporting
  - Memory usage tracking per execution
  - Configurable per-script memory limits

- **Sandbox Security Model**
  - Blocked module imports (os, sys, subprocess, importlib)
  - Prevented eval/exec execution
  - Dynamic attribute access blocking (getattr)
  - File path traversal prevention
  - Configurable file access whitelist

- **Resource Management**
  - Execution timeout support (configurable)
  - Process cleanup and orphan prevention
  - Graceful termination of child processes
  - Resource deallocation on execution end

- **Public API**
  - `CommandEngine` class for execution control
  - `ScriptRunner` class for Python-specific execution
  - `ExecutionResult` struct for output/status/metadata
  - Error handling and status reporting

### Verification Status

**Test Suite Results:**
- Total tests: 27
- Passed: 27
- Failed: 0
- Security violations: 0

**Verified Protections:**
- ✅ IPC message delivery and validation
- ✅ Child Python script execution
- ✅ Sandbox import restrictions (blocked os, sys, subprocess)
- ✅ File path traversal prevention
- ✅ Memory limit enforcement via Job Objects
- ✅ Timeout enforcement
- ✅ Process cleanup (no orphans)
- ✅ Privilege level maintained (no escalation)

### Platform Support

- **Windows**: 7 SP1 and later (fully tested on Windows 10+)
- **Architecture**: x64 (primary), x86 available
- **Python**: 3.9, 3.10, 3.11 tested

### Known Limitations

- Windows only (macOS and Linux in v1.0)
- No JavaScript/Node.js support (planned v1.0)
- No remote execution (local only)
- Sandbox not designed for adversarial code
- File access restricted to configured paths only
- No cryptographic communication (local IPC only)

### Installation

See [INSTALLATION.md](INSTALLATION.md) for download and setup.

### Documentation

- [README.md](README.md) — Overview and quick start
- [API.md](API.md) — Complete public API reference
- [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md) — Integration guide
- [SECURITY.md](SECURITY.md) — Security model and limitations
- [examples/](examples/) — Code examples and use cases

### What's Not Included

- Internal source code (closed-source SDK)
- Build system files (CMake, Visual Studio projects)
- Test suite (development only)
- Debug symbols (removed for release)
- Private documentation (internal reports)

### Performance

- Process creation: 50-200 ms (initial), <10 ms (subsequent)
- Memory overhead: ~5-10 MB parent, configurable child minimum
- Timeout precision: ±50 ms (OS dependent)
- Output capture: Up to 10 MB per execution

### Breaking Changes

None (first release).

### Deprecations

None (first release).

### Bug Fixes

None (first release).

## Planned Future Releases

### v0.2 (Q3 2026)
- Extended script execution modes
- Better error messages
- Configuration profiles
- Performance improvements

### v0.5 (Q4 2026)
- macOS validation
- Linux beta support
- Real-time monitoring API
- Execution history

### v1.0 (June 24, 2026)
- Full cross-platform support (Windows, macOS, Linux)
- Plugin architecture
- JavaScript/Node.js support
- Third-party security audit
- Production-ready stability

See [ROADMAP.md](ROADMAP.md) for detailed future plans.

## Security

For security issues, see [SECURITY.md](SECURITY.md#reporting-security-issues).

This is a **beta release**. Security validation is ongoing. Third-party audit is planned for v1.0.

## License

See [PROPRIETARY_LICENSE.md](PROPRIETARY_LICENSE.md).

---

**Current**: UCE v0.1 Beta
**Next**: UCE v0.2 (Q3 2026)
**Target**: UCE v1.0 production (June 24, 2026)

Thank you for testing UCE v0.1 Beta!
