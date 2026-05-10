# UCE Public API

## Overview

UCE v0.1 Beta exposes a public C++ API for executing commands and Python scripts in isolated child processes with resource constraints.

This API is designed for host applications to integrate UCE as a closed-source SDK.

## Key API Concepts

- `CommandEngine` — main host-facing execution interface
- `ScriptRunner` — convenient Python execution wrapper
- `ExecutionResult` — execution output, status, and metadata
- `ExecutionStatus` — result classification for success and failures

## Public API Classes

### CommandEngine

The `CommandEngine` object provides a simple interface to execute code in an isolated child process.

```cpp
#include <uce/CommandEngine.h>

uce::CommandEngine engine;
engine.SetMemoryLimit(256 * 1024 * 1024); // 256 MB
engine.SetTimeout(5000);                   // 5 seconds

uce::ExecutionResult result = engine.ExecutePython("print('Hello from UCE')");
```

Common methods:

- `SetMemoryLimit(size_t bytes)` — configure memory ceiling for the child process
- `SetTimeout(uint32_t milliseconds)` — configure execution timeout
- `ExecutePython(const std::string& script)` — execute Python code
- `ExecuteCommand(const std::string& command)` — execute command or shell-style string
- `GetStatus()` — retrieve the last execution status

### ScriptRunner

`ScriptRunner` is optimized for Python script execution and sandbox handling.

```cpp
#include <uce/ScriptRunner.h>

uce::ScriptRunner runner;
runner.SetMemoryLimit(128 * 1024 * 1024);
runner.SetTimeout(8000);
runner.AllowFileAccess("C:\\data\\scripts");

uce::ExecutionResult result = runner.Execute("print('Run inside UCE sandbox')");
```

Common methods:

- `SetMemoryLimit(size_t bytes)`
- `SetTimeout(uint32_t milliseconds)`
- `AllowFileAccess(const std::string& path)` — whitelist a directory for file access
- `Execute(const std::string& python_code)` — run Python code in sandbox
- `WasSandboxed()` — verify if the execution used sandbox restrictions
- `GetRawOutput()` — obtain full captured output string

## Execution Result

The `ExecutionResult` structure contains output and metadata.

```cpp
struct ExecutionResult {
    ExecutionStatus status;
    std::string stdout_output;
    std::string stderr_output;
    uint64_t memory_used;
    uint32_t exit_code;
    std::string error_message;
};
```

### ExecutionStatus

```cpp
enum class ExecutionStatus {
    Success,
    Timeout,
    MemoryLimitExceeded,
    SandboxViolation,
    ProcessError,
    IPC_Error,
    Unknown
};
```

## Example Usage

### Command Execution

```cpp
uce::CommandEngine engine;
engine.SetMemoryLimit(256 * 1024 * 1024);
engine.SetTimeout(5000);

uce::ExecutionResult result = engine.ExecuteCommand("python -c \"print('Hello')\"");
```

### Python Script Execution

```cpp
uce::CommandEngine engine;
engine.SetMemoryLimit(256 * 1024 * 1024);
engine.SetTimeout(10000);

std::string script = R"(
print('UCE Python Execution')
for i in range(3):
    print(i)
)";

uce::ExecutionResult result = engine.ExecutePython(script);
```

### Isolated Execution

```cpp
for (int i = 0; i < 3; ++i) {
    uce::CommandEngine engine;
    engine.SetMemoryLimit(128 * 1024 * 1024);
    engine.SetTimeout(5000);
    
    uce::ExecutionResult result = engine.ExecutePython("print('Execution " + std::to_string(i) + "')");
}
```

### Sandbox Violation Handling

```cpp
uce::ScriptRunner runner;
runner.SetMemoryLimit(128 * 1024 * 1024);
runner.SetTimeout(5000);

uce::ExecutionResult result = runner.Execute("import os\nprint(os.getcwd())");
if (result.status == uce::ExecutionStatus::SandboxViolation) {
    std::cerr << "Sandbox violation: " << result.error_message << std::endl;
}
```

### Memory Limit Behavior

```cpp
uce::ScriptRunner runner;
runner.SetMemoryLimit(64 * 1024 * 1024);
runner.SetTimeout(5000);

std::string script = R"(
# Attempt a large allocation
arr = [0] * (50 * 1024 * 1024)
print('Allocated large list')
)";

uce::ExecutionResult result = runner.Execute(script);
if (result.status == uce::ExecutionStatus::MemoryLimitExceeded) {
    std::cout << "Memory limit enforced" << std::endl;
}
```

## Public API Notes

- `ExecutionResult` should always be checked after each call.
- `SetMemoryLimit(0)` disables memory enforcement.
- `SetTimeout(0)` disables automatic timeout.
- `CommandEngine` and `ScriptRunner` are not thread-safe; use separate instances per thread.
- API names and behavior are public-facing and stable for v0.1 Beta.

## Usage Patterns

### Simple Execution Pattern

```cpp
uce::CommandEngine engine;
engine.SetMemoryLimit(256 * 1024 * 1024);
engine.SetTimeout(5000);

uce::ExecutionResult result = engine.ExecutePython("print('Hello UCE')");
if (result.status == uce::ExecutionStatus::Success) {
    std::cout << result.stdout_output << std::endl;
}
```

### Safe Script Execution Pattern

```cpp
uce::ScriptRunner runner;
runner.SetMemoryLimit(128 * 1024 * 1024);
runner.SetTimeout(8000);
runner.AllowFileAccess("C:\\data\\scripts");

uce::ExecutionResult result = runner.Execute("print('Sandboxed execution')");
```

## Platform Notes

- Windows only in v0.1 Beta
- Requires Python 3.9+ for Python execution
- Uses process-level isolation and IPC for result delivery

## See Also

- [README.md](README.md)
- [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md)
- [INSTALLATION.md](INSTALLATION.md)
- [SECURITY.md](SECURITY.md)
- [examples/](examples/)
