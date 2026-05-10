# UCE Integration Guide

This guide shows how to integrate UCE v0.1 Beta into your Windows C++ application.

## Prerequisites

- Windows 7 SP1 or later
- Visual Studio 2015 or later (CMake 3.10+ compatible)
- Python 3.9+ (for Python script execution)
- C++17 or later

## Installation

### 1. Download and Extract

1. Download UCE v0.1 Beta release package
2. Extract to your project directory:

```
YourProject/
├── vendor/
│   └── uce_v0.1_beta/
│       ├── bin/
│       │   ├── uce_command_engine.dll
│       │   └── uce_script_runner.exe
│       ├── lib/
│       │   ├── uce_command_engine.lib
│       │   └── uce_script_runner.lib
│       └── include/
│           └── uce/
│               ├── CommandEngine.h
│               ├── ScriptRunner.h
│               └── ExecutionResult.h
```

### 2. Configure CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyApp)

set(CMAKE_CXX_STANDARD 17)

# Add UCE
set(UCE_ROOT "vendor/uce_v0.1_beta")

# Include headers
include_directories("${UCE_ROOT}/include")

# Link library
link_directories("${UCE_ROOT}/lib")

# Your executable
add_executable(myapp main.cpp)

# Link UCE
target_link_libraries(myapp uce_command_engine)

# Copy DLLs to output (Windows)
add_custom_command(TARGET myapp POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    "${UCE_ROOT}/bin" "$<TARGET_FILE_DIR:myapp>"
    COMMENT "Copying UCE binaries..."
)
```

### 3. Visual Studio Project Setup (Alternative)

If not using CMake:

1. Right-click project → Properties
2. **VC++ Directories** → **Include Directories**: Add `vendor\uce_v0.1_beta\include`
3. **VC++ Directories** → **Library Directories**: Add `vendor\uce_v0.1_beta\lib`
4. **Linker** → **Input** → **Additional Dependencies**: Add `uce_command_engine.lib`
5. Copy `uce_*.dll` files to your output directory

## Basic Integration

### Minimal Example

```cpp
#include <iostream>
#include <uce/CommandEngine.h>

int main() {
    // Create engine
    uce::CommandEngine engine;
    
    // Set constraints
    engine.SetMemoryLimit(256 * 1024 * 1024);  // 256 MB
    engine.SetTimeout(5000);                    // 5 seconds
    
    // Execute code
    uce::ExecutionResult result = engine.ExecutePython(
        "print('Hello from UCE')"
    );
    
    // Check result
    if (result.status == uce::ExecutionStatus::Success) {
        std::cout << "Success: " << result.stdout_output << std::endl;
    } else {
        std::cout << "Error: " << result.error_message << std::endl;
    }
    
    return 0;
}
```

### Host Application Flow

1. **Create engine instance** → `CommandEngine engine;`
2. **Configure constraints** → `engine.SetMemoryLimit(...)`, `engine.SetTimeout(...)`
3. **Prepare execution** → Validate input, prepare script
4. **Execute** → `ExecutionResult result = engine.ExecutePython(script);`
5. **Handle result** → Check status, process output, handle errors
6. **Clean up** → Automatic; engine destructor terminates child processes

### Integration Pattern

```cpp
class ScriptExecutor {
private:
    uce::CommandEngine engine;
    
public:
    // Execute user-provided script safely
    std::string ExecuteUserScript(const std::string& user_script) {
        // Validate script length
        if (user_script.length() > 1000000) {
            throw std::runtime_error("Script too large");
        }
        
        // Set safe defaults
        engine.SetMemoryLimit(128 * 1024 * 1024);  // 128 MB
        engine.SetTimeout(10000);                   // 10 seconds
        
        // Execute
        uce::ExecutionResult result = engine.ExecutePython(user_script);
        
        // Handle result
        switch (result.status) {
            case uce::ExecutionStatus::Success:
                return result.stdout_output;
            case uce::ExecutionStatus::MemoryLimitExceeded:
                throw std::runtime_error("Script exceeded memory limit");
            case uce::ExecutionStatus::Timeout:
                throw std::runtime_error("Script execution timed out");
            case uce::ExecutionStatus::SandboxViolation:
                throw std::runtime_error("Blocked operation attempted");
            default:
                throw std::runtime_error("Execution error: " + result.error_message);
        }
    }
};
```

## Deployment

### Package Structure

When distributing your application with UCE:

```
YourApp-v1.0-installer/
├── bin/
│   ├── YourApp.exe
│   ├── uce_command_engine.dll
│   └── uce_script_runner.exe
├── data/
├── README.txt
└── LICENSE.txt
```

### DLL Deployment

**Required files** for runtime:
- `uce_command_engine.dll` (main library)
- `uce_script_runner.exe` (child process executable)

Both must be in the same directory as your application `.exe` or in system PATH.

### Versioning

Always use matching versions:
- Link against `uce_command_engine.lib` v0.1
- Runtime requires `uce_command_engine.dll` v0.1
- Child process requires `uce_script_runner.exe` v0.1

Mismatch will result in IPC errors.

## Error Handling

### Common Issues

**Issue**: "Cannot find uce_command_engine.dll"
- **Solution**: Ensure DLL is in same directory as .exe or system PATH

**Issue**: IPC_Error in execution
- **Solution**: Verify all DLL files are from same v0.1 Beta release

**Issue**: Timeout not enforced
- **Solution**: Check Windows Job Object support (Windows 7 SP1+)

**Issue**: Memory limit not enforced
- **Solution**: Ensure memory is 8 MB or greater; Windows enforces minimum

### Graceful Degradation

```cpp
uce::ExecutionResult result = engine.ExecutePython(script);

// Graceful handling of each error
if (result.status == uce::ExecutionStatus::Success) {
    // Proceed with result
    ProcessOutput(result.stdout_output);
} else if (result.status == uce::ExecutionStatus::Timeout) {
    // Log timeout, suggest user optimize script
    LogWarning("Script execution timed out after " << 
               engine.GetTimeout() << "ms");
} else if (result.status == uce::ExecutionStatus::MemoryLimitExceeded) {
    // Log memory issue
    LogWarning("Script exceeded memory limit: " << 
               result.memory_used << " bytes used");
} else {
    // Generic error
    LogError("Script execution failed: " << result.error_message);
}
```

## Performance Considerations

### Process Overhead

- **First execution**: 50-200 ms (child process startup)
- **Subsequent**: <10 ms (if using connection pooling)
- **Output capture**: Minimal overhead

### Memory Usage

- **Parent process**: +5-10 MB for engine
- **Child process**: Configurable minimum 8 MB
- **Per execution**: Overhead ~2 MB + user script memory

### Optimization Tips

1. **Reuse engine instances** — Create once, execute multiple times
2. **Set realistic timeouts** — Match expected script duration
3. **Batch operations** — Execute multiple scripts in sequence
4. **Monitor memory** — Log peak memory usage for tuning

## Security Considerations

### Closed-Source SDK

This is a binary-only SDK. Internal implementation details are proprietary.

### Sandbox Limitations

The sandbox prevents:
- Accidental API misuse
- Cooperative script errors
- Unauthorized file access

The sandbox does NOT prevent:
- Determined attackers with OS knowledge
- Python interpreter exploits
- OS kernel vulnerabilities

**Never run untrusted adversarial code** in production without OS-level isolation (containers, VMs).

### User Input Validation

Always validate user-provided scripts:

```cpp
bool IsScriptSafe(const std::string& script) {
    // Check size
    if (script.empty() || script.size() > 1000000) {
        return false;
    }
    
    // Check for suspicious patterns (optional)
    // This is not a security guarantee, just basic validation
    
    return true;
}
```

## Beta Status Notes

UCE v0.1 Beta:
- May have undiscovered edge cases
- Not recommended for production untrusted code
- API may change before v1.0 (June 24, 2026)
- Pre-release security validation ongoing

## Support and Examples

- See [examples/](../examples/) for more code samples
- See [API.md](../API.md) for complete API reference
- See [docs/FAQ.md](../docs/FAQ.md) for common questions
- See [SECURITY.md](../SECURITY.md) for security model details

---

Ready to integrate? Start with the [Minimal Example](#minimal-example) above and adapt to your use case.

For v1.0 and later versions, additional features will be available (async execution, plugin system, cross-platform support).
