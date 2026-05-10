# UCE Development Roadmap

## Current Release

### v0.1 Beta (Now)

- Windows-only command and script execution framework
- Process isolation with OS-level memory enforcement
- Python script execution in child processes
- Sandbox with restricted imports
- File access controls
- IPC communication
- 27 verified security and functionality tests
- Closed-source binary SDK

**Target audience**: Windows developers evaluating isolated command execution for their applications.

## Planned Releases

### v0.2 (Q3 2026)

**Focus**: Extended execution modes and better error handling

- [ ] Extended script execution (Batch/PowerShell)
- [ ] Improved error messages and diagnostics
- [ ] Configuration profiles for different isolation levels
- [ ] Performance improvements for rapid script execution
- [ ] Better timeout handling and graceful termination

### v0.5 (Q4 2026)

**Focus**: Cross-platform validation and enhanced monitoring

- [ ] macOS validation and testing
- [ ] Linux beta support (Ubuntu 20.04+)
- [ ] Real-time execution monitoring API
- [ ] Extended resource limits (disk I/O, CPU affinity)
- [ ] Execution history and logging
- [ ] Beta plugin system foundation

### v1.0 (June 24, 2026)

**Focus**: Production-ready multi-platform framework with plugin ecosystem

- [ ] Full Windows, macOS, Linux support
- [ ] Plugin architecture (custom execution backends)
- [ ] JavaScript/Node.js execution backend
- [ ] Commercial licensing model
- [ ] Enterprise support options
- [ ] Performance optimizations (25%+ throughput improvement)
- [ ] Advanced isolation metrics and reporting
- [ ] Cloud execution backend (preview)

## Features in Evaluation

- **Remote execution**: Execute scripts on remote systems (v1.x)
- **Container integration**: Direct Docker/container execution (v1.x)
- **Distributed execution**: Multi-machine script execution (v2.0+)
- **Custom sandboxes**: User-defined isolation policies (v1.0+)
- **Language packs**: Go, Rust, C# backends (v1.x+)

## Deprecation Policy

Versions are supported for 18 months after release:

- v0.1 support ends: December 2027
- v0.2 support ends: June 2028
- v1.0 support ends: December 2027 (initial release) → June 2028

## How to Stay Updated

- Watch this repository for releases
- Check [CHANGELOG.md](CHANGELOG.md) for changes
- See [RELEASE_NOTES.md](RELEASE_NOTES.md) for v0.1 Beta details

---

**Next milestone**: v1.0 production release June 24, 2026.
