# Python Script Execution Example

This example demonstrates executing complete Python scripts with data processing and calculations.

## Scenario

Execute a Python data processing script that calculates statistics from a dataset, demonstrating realistic Python usage within UCE isolation.

## Code

```cpp
#include <iostream>
#include <string>
#include <uce/CommandEngine.h>

int main() {
    std::cout << "=== UCE Python Script Execution ===" << std::endl;
    std::cout << std::endl;
    
    uce::CommandEngine engine;
    engine.SetMemoryLimit(512 * 1024 * 1024);  // 512 MB
    engine.SetTimeout(10000);                   // 10 seconds
    
    // Python script for data analysis
    std::string python_script = R"(
import json

# Sample dataset
data = [10, 15, 20, 25, 30, 35, 40, 45, 50]

# Calculate statistics
def calculate_stats(values):
    n = len(values)
    mean = sum(values) / n
    variance = sum((x - mean) ** 2 for x in values) / n
    std_dev = variance ** 0.5
    
    return {
        'count': n,
        'sum': sum(values),
        'mean': mean,
        'min': min(values),
        'max': max(values),
        'variance': variance,
        'std_dev': std_dev
    }

stats = calculate_stats(data)

# Output as JSON
print("Statistics:")
print(json.dumps(stats, indent=2))
)";
    
    std::cout << "Executing Python data analysis script..." << std::endl;
    uce::ExecutionResult result = engine.ExecutePython(python_script);
    
    std::cout << std::endl;
    
    if (result.status == uce::ExecutionStatus::Success) {
        std::cout << "--- Results ---" << std::endl;
        std::cout << result.stdout_output << std::endl;
        
        std::cout << "--- Execution Details ---" << std::endl;
        std::cout << "Status: SUCCESS" << std::endl;
        std::cout << "Exit code: " << result.exit_code << std::endl;
        std::cout << "Peak memory: " << (result.memory_used / 1024 / 1024) << " MB" << std::endl;
    } else {
        std::cout << "Error: " << result.error_message << std::endl;
        if (!result.stderr_output.empty()) {
            std::cout << "Details: " << result.stderr_output << std::endl;
        }
    }
    
    return 0;
}
```

## Script Capabilities Demonstrated

✅ **Standard library usage**: `json` module for data serialization

✅ **Data structures**: Lists and dictionaries for data handling

✅ **Mathematical operations**: Variance and standard deviation calculations

✅ **Built-in functions**: `sum()`, `min()`, `max()`, `len()`

✅ **String formatting**: Formatted output with precision

✅ **Control flow**: Loops and conditionals

❌ **Blocked operations**: `os`, `sys`, `subprocess` imports (intentionally blocked)

## Running

```cmd
python_execution.exe
```

Expected output:

```
=== UCE Python Script Execution ===

Executing Python data analysis script...

--- Results ---
Statistics:
{
  "count": 9,
  "sum": 270,
  "mean": 30.0,
  "min": 10,
  "max": 50,
  "variance": 166.66666666666666,
  "std_dev": 12.909944487358055
}

--- Execution Details ---
Status: SUCCESS
Exit code: 0
Peak memory: 18 MB
```

## Advanced Python Features Supported

- ✅ Functions and loops
- ✅ List comprehensions
- ✅ Lambda expressions
- ✅ Built-in modules (json, math, random, datetime, etc.)
- ✅ Exception handling (try/except)
- ✅ String manipulation and formatting
- ✅ Numpy/Pandas (if installed in host Python)

## Blocked Operations

The following are intentionally blocked by the sandbox:

- ❌ `import os` — No OS-level access
- ❌ `import sys` — No system information
- ❌ `import subprocess` — No process creation
- ❌ `exec()` or `eval()` — No dynamic code execution
- ❌ `__import__()` — No dynamic imports
- ❌ `getattr()` for attribute tricks — No attribute magic tricks

## Error Handling

```cpp
if (result.status != uce::ExecutionStatus::Success) {
    switch (result.status) {
        case uce::ExecutionStatus::SandboxViolation:
            std::cerr << "Sandbox block: " << result.error_message << std::endl;
            // Script tried blocked operation
            break;
        case uce::ExecutionStatus::Timeout:
            std::cerr << "Script execution timed out" << std::endl;
            break;
        case uce::ExecutionStatus::MemoryLimitExceeded:
            std::cerr << "Script exceeded memory limit" << std::endl;
            break;
        default:
            std::cerr << "Error: " << result.error_message << std::endl;
    }
}
```

## Performance Notes

- First execution: 50-200 ms (Python startup)
- Subsequent executions: <10 ms (if process reused)
- Memory overhead: ~10-20 MB per execution
- Output capture: Up to 10 MB per execution

## Next Steps

- See [isolated_execution.md](isolated_execution.md) for running multiple scripts
- See [sandbox_violation.md](sandbox_violation.md) for security constraints
- See [memory_limit.md](memory_limit.md) for memory enforcement
- See [API.md](../API.md) for complete API reference

---

For more Python examples and integration patterns, see [EMBEDDING_GUIDE.md](../EMBEDDING_GUIDE.md).
