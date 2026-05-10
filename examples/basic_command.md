# Basic Command Execution Example

This example demonstrates basic isolated command execution using the UCE SDK.

## Scenario

Execute a simple Python command in an isolated child process with memory and timeout constraints.

## Code

```cpp
#include <iostream>
#include <uce/CommandEngine.h>

int main() {
    std::cout << "=== UCE Basic Command Execution ===" << std::endl;
    
    // Create engine instance
    uce::CommandEngine engine;
    
    // Configure constraints
    const size_t MEMORY_LIMIT = 256 * 1024 * 1024;  // 256 MB
    const uint32_t TIMEOUT_MS = 5000;               // 5 seconds
    
    engine.SetMemoryLimit(MEMORY_LIMIT);
    engine.SetTimeout(TIMEOUT_MS);
    
    std::cout << "Memory limit: " << (MEMORY_LIMIT / 1024 / 1024) << " MB" << std::endl;
    std::cout << "Timeout: " << TIMEOUT_MS << " ms" << std::endl;
    std::cout << std::endl;
    
    // Execute Python code
    std::string python_code = R"(
import sys
print("Hello from isolated Python!")
print(f"Python version: {sys.version}")
print("This code runs in a sandboxed child process")
)";
    
    std::cout << "Executing Python code..." << std::endl;
    uce::ExecutionResult result = engine.ExecutePython(python_code);
    std::cout << std::endl;
    
    // Check result
    std::cout << "Execution status: ";
    
    switch (result.status) {
        case uce::ExecutionStatus::Success:
            std::cout << "SUCCESS" << std::endl;
            break;
        case uce::ExecutionStatus::Timeout:
            std::cout << "TIMEOUT" << std::endl;
            break;
        case uce::ExecutionStatus::MemoryLimitExceeded:
            std::cout << "MEMORY LIMIT EXCEEDED" << std::endl;
            break;
        case uce::ExecutionStatus::SandboxViolation:
            std::cout << "SANDBOX VIOLATION" << std::endl;
            break;
        default:
            std::cout << "ERROR" << std::endl;
    }
    
    // Display output
    if (!result.stdout_output.empty()) {
        std::cout << "\n--- Output ---" << std::endl;
        std::cout << result.stdout_output << std::endl;
    }
    
    if (!result.stderr_output.empty()) {
        std::cout << "\n--- Errors ---" << std::endl;
        std::cout << result.stderr_output << std::endl;
    }
    
    // Display metadata
    std::cout << "\n--- Metadata ---" << std::endl;
    std::cout << "Exit code: " << result.exit_code << std::endl;
    std::cout << "Peak memory: " << (result.memory_used / 1024 / 1024) << " MB" << std::endl;
    
    return 0;
}
```

## Build Instructions

Using CMake:

```cmake
cmake_minimum_required(VERSION 3.10)
project(UCE_Example_Basic)

set(CMAKE_CXX_STANDARD 17)

# Add UCE
include_directories("C:/opt/uce/include")
link_directories("C:/opt/uce/lib")

add_executable(basic_example basic_command.cpp)
target_link_libraries(basic_example uce_command_engine)
```

Or with Visual Studio:

1. Create new C++ console project
2. Properties → VC++ Directories:
   - Include Directories: `C:\opt\uce\include`
   - Library Directories: `C:\opt\uce\lib`
3. Linker → Input: Add `uce_command_engine.lib`
4. Copy DLLs from `C:\opt\uce\bin` to build output

## Running

```cmd
basic_example.exe
```

Expected output:

```
=== UCE Basic Command Execution ===
Memory limit: 256 MB
Timeout: 5000 ms

Executing Python code...

Execution status: SUCCESS

--- Output ---
Hello from isolated Python!
Python version: 3.9.x ...
This code runs in a sandboxed child process

--- Metadata ---
Exit code: 0
Peak memory: 15 MB
```

## Key Points

- **Engine creation**: `CommandEngine` manages execution
- **Constraints**: Memory limit (256 MB) and timeout (5 seconds)
- **Execution**: `ExecutePython()` runs code in isolated process
- **Result checking**: Always check `result.status` before using output
- **Metadata**: `memory_used` shows peak memory during execution
- **Cleanup**: Automatic when `ExecutionResult` is destroyed

## Next Steps

- See [python_execution.md](python_execution.md) for more advanced Python
- See [isolated_execution.md](isolated_execution.md) for multiple executions
- See [sandbox_violation.md](sandbox_violation.md) for security examples
- See [memory_limit.md](memory_limit.md) for memory enforcement examples

See [API.md](../API.md) for complete API reference.
