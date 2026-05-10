# Universal Command Engine (UCE) v0.1 Beta

UCE is a closed-source, production-focused command execution framework for Windows that provides isolated and sandboxed script execution with memory and resource constraints.

## Status

**UCE v0.1 Beta** — Closed-source public SDK release for early testing and integration.

This is a **beta release** intended for evaluation and integration testing. Full v1.0 is targeted for **June 24, 2026**.

## What is UCE?

UCE enables secure execution of commands and scripts (Python, shell) in isolated child processes with:

- **Process isolation**: Child processes run under controlled parent supervision
- **Memory enforcement**: OS-level memory limits via Windows Job Objects
- **Sandbox model**: Blocked imports (os, sys, subprocess), restricted file access, blocked eval/exec
- **IPC communication**: Secure inter-process communication with metadata flow
- **Resource monitoring**: Real-time process memory and resource tracking
- **Timeout protection**: Configurable execution timeouts

## Closed-Source SDK

This is a **binary SDK release**. Internal source code is not included. Integration uses:
- Pre-built binaries (`uce_command_engine.dll`, `uce_script_runner.exe`)
- Public headers defining the API
- Documentation and examples

## Verified Beta Status

Testing results on v0.1 Beta:
- **Tests passed**: 27
- **Tests failed**: 0
- **Security violations**: 0

Verified protections:
- ✅ IPC message delivery and isolation
- ✅ Python script execution in child processes
- ✅ Sandbox import restrictions (os, sys, subprocess blocked)
- ✅ File path traversal prevention
- ✅ Memory limit enforcement via OS Job Objects
- ✅ Process cleanup and resource deallocation
- ✅ Timeout handling

## Quick Start

### Installation

1. Download the UCE v0.1 Beta release package
2. Extract binaries and headers
3. Link `uce_command_engine.lib` in your project
4. Include `<uce/CommandEngine.h>`

### Basic Usage

```cpp
#include <uce/CommandEngine.h>

// Execute an isolated command
ScriptRunner runner;
runner.SetMemoryLimit(256 * 1024 * 1024); // 256 MB
runner.SetTimeout(5000); // 5 seconds

ExecutionResult result = runner.Execute("print('Hello from isolated Python')");
if (result.status == ExecutionStatus::Success) {
    std::cout << result.stdout_output << std::endl;
}
```

See [API.md](API.md) for detailed API documentation and [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md) for integration patterns.

## Features

- **Multiple execution modes**: Command execution, Python scripts, shell commands
- **Isolation guarantees**: Process and memory isolation at OS level
- **Resource constraints**: Memory limits, timeout, CPU isolation via Job Objects
- **Security model**: Sandbox restrictions on dangerous operations
- **Host application integration**: Easy embedding in C++ applications
- **Real-time monitoring**: Memory usage tracking, execution status

## Security Model Overview

UCE implements a **defense-in-depth approach**:

1. **Process-level isolation**: Child processes are isolated from parent
2. **OS memory enforcement**: Windows Job Object memory limits prevent out-of-memory attacks
3. **Sandbox restrictions**: Blocked imports prevent access to dangerous APIs (os.system, subprocess, etc.)
4. **Path restrictions**: File access limited to specified directories
5. **IPC security**: Metadata and communication validated per execution

**Not a security boundary for untrusted code.** For true sandboxing of adversarial code, use OS-level isolation (containers, VMs).

## Known Limitations

- Windows only (beta release)
- Python 3.9+ required for Python script execution
- No JavaScript/Node.js support (planned for future releases)
- No remote execution (local process execution only)
- Sandbox not designed for adversarial untrusted code
- File access restricted to configured paths

## v1.0 Roadmap

**Targeted June 24, 2026:**
- Linux and macOS validation
- Extended timeout and monitoring options
- Plugin architecture
- JavaScript/Node.js backend support
- Enterprise licensing model
- Performance optimizations

## 📥 Download UCE

[![Download UCE v0.1 BETA Windows x64](https://img.shields.io/badge/UCE_v0.1_Beta_Windows_x64-Download-blue?style=for-the-badge&logo=windows)](https://github.com/Sanji1-Owner/Universal-Command-Engine/raw/main/UCE_v0.1_Beta_Windows_x64.zip)


## Documentation

- [API.md](API.md) — Public API reference
- [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md) — Integration and usage patterns
- [INSTALLATION.md](INSTALLATION.md) — Installation and setup
- [SECURITY.md](SECURITY.md) — Security model and reporting
- [CHANGELOG.md](CHANGELOG.md) — Changes and fixes in this release
- [ROADMAP.md](ROADMAP.md) — Future releases and features
- [docs/](docs/) — Architecture overview and FAQ

## Support

For integration questions, see [docs/FAQ.md](docs/FAQ.md).

For security issues, see [SECURITY.md](SECURITY.md).

## License

See [PROPRIETARY_LICENSE.md](PROPRIETARY_LICENSE.md) for licensing terms.

---

**UCE v0.1 Beta** — Closed-source SDK for Windows. v1.0 targeted June 24, 2026.
