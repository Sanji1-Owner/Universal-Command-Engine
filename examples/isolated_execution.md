# Isolated Execution Example

This example demonstrates executing multiple scripts in isolated processes while maintaining resource constraints and handling results.

## Scenario

Execute a series of Python scripts, each in a separate isolated child process, demonstrating how to manage multiple concurrent executions.

## Code

```cpp
#include <iostream>
#include <vector>
#include <uce/CommandEngine.h>

struct ExecutionTask {
    std::string name;
    std::string script;
    size_t memory_limit;
    uint32_t timeout;
};

int main() {
    std::cout << "=== UCE Isolated Execution ===" << std::endl;
    std::cout << std::endl;
    
    // Define multiple tasks
    std::vector<ExecutionTask> tasks = {
        {
            "Task 1: String Processing",
            R"(
text = "Hello, World!"
words = text.split()
print(f"Found {len(words)} words: {words}")
)",
            128 * 1024 * 1024,  // 128 MB
            5000                 // 5 seconds
        },
        {
            "Task 2: List Operations",
            R"(
numbers = [1, 2, 3, 4, 5]
squared = [x ** 2 for x in numbers]
print(f"Original: {numbers}")
print(f"Squared: {squared}")
)",
            128 * 1024 * 1024,
            5000
        },
        {
            "Task 3: Dictionary Processing",
            R"(
person = {'name': 'Alice', 'age': 30, 'city': 'NYC'}
for key, value in person.items():
    print(f"{key}: {value}")
)",
            128 * 1024 * 1024,
            5000
        }
    };
    
    // Execute each task in isolation
    int task_count = 0;
    for (const auto& task : tasks) {
        task_count++;
        std::cout << "--- " << task.name << " ---" << std::endl;
        
        // Create fresh engine for each task
        uce::CommandEngine engine;
        engine.SetMemoryLimit(task.memory_limit);
        engine.SetTimeout(task.timeout);
        
        // Execute
        uce::ExecutionResult result = engine.ExecutePython(task.script);
        
        // Report results
        std::cout << "Status: ";
        
        if (result.status == uce::ExecutionStatus::Success) {
            std::cout << "SUCCESS" << std::endl;
            std::cout << "Output:" << std::endl;
            std::cout << result.stdout_output;
            std::cout << "Memory used: " << (result.memory_used / 1024) << " KB" << std::endl;
        } else {
            std::cout << "FAILED" << std::endl;
            std::cout << "Reason: " << result.error_message << std::endl;
        }
        
        std::cout << std::endl;
    }
    
    std::cout << "=== Summary ===" << std::endl;
    std::cout << "Executed " << task_count << " isolated tasks" << std::endl;
    std::cout << "Each task ran in separate child process" << std::endl;
    std::cout << "All resources cleaned up automatically" << std::endl;
    
    return 0;
}
```

## Key Concepts

### Process Isolation

Each execution creates a **separate child process**:

```
Parent Process (Your Application)
    ├─ Child Process 1 (Task 1)
    ├─ Child Process 2 (Task 2)
    └─ Child Process 3 (Task 3)
```

- Each child runs independently
- No sharing of memory or state between tasks
- IPC returns results to parent
- Child cleanup is automatic

### Resource Management

```cpp
// Each task can have different constraints
uce::CommandEngine engine;
engine.SetMemoryLimit(64 * 1024 * 1024);    // 64 MB for lightweight task
engine.SetTimeout(2000);                     // 2 second timeout

// Separate engine = separate child process
uce::CommandEngine engine2;
engine2.SetMemoryLimit(512 * 1024 * 1024);  // 512 MB for heavy task
engine2.SetTimeout(30000);                   // 30 second timeout
```

### Cleanup Guarantee

```cpp
{
    uce::CommandEngine engine;
    uce::ExecutionResult result = engine.ExecutePython("...");
    
    // Use result here
    std::cout << result.stdout_output;
    
}  // <-- engine destroyed here
   // <-- child process terminated automatically
   // <-- all resources released
```

## Running

```cmd
isolated_execution.exe
```

Expected output:

```
=== UCE Isolated Execution ===

--- Task 1: String Processing ---
Status: SUCCESS
Output:
Found 2 words: ['Hello,', 'World!']
Memory used: 12288 KB

--- Task 2: List Operations ---
Status: SUCCESS
Output:
Original: [1, 2, 3, 4, 5]
Squared: [1, 4, 9, 16, 25]
Memory used: 11520 KB

--- Task 3: Dictionary Processing ---
Status: SUCCESS
Output:
name: Alice
age: 30
city: NYC
Memory used: 12032 KB

=== Summary ===
Executed 3 isolated tasks
Each task ran in separate child process
All resources cleaned up automatically
```

## Advanced Patterns

### Sequential Execution with State Management

```cpp
struct ExecutionState {
    int success_count;
    int failure_count;
    size_t total_memory_used;
};

ExecutionState state = {0, 0, 0};

for (const auto& task : tasks) {
    uce::CommandEngine engine;
    uce::ExecutionResult result = engine.ExecutePython(task.script);
    
    if (result.status == uce::ExecutionStatus::Success) {
        state.success_count++;
        state.total_memory_used += result.memory_used;
    } else {
        state.failure_count++;
    }
}

std::cout << "Successes: " << state.success_count << std::endl;
std::cout << "Failures: " << state.failure_count << std::endl;
std::cout << "Total memory: " << (state.total_memory_used / 1024 / 1024) << " MB" << std::endl;
```

### Dynamic Task Generation

```cpp
std::vector<std::string> scripts;

// Generate scripts dynamically
for (int i = 1; i <= 10; i++) {
    scripts.push_back(
        "for x in range(" + std::to_string(i) + "):\n"
        "    print(f'Iteration {x}')"
    );
}

// Execute all
for (const auto& script : scripts) {
    uce::CommandEngine engine;
    uce::ExecutionResult result = engine.ExecutePython(script);
    // Process results
}
```

### Error Recovery

```cpp
for (const auto& task : tasks) {
    uce::CommandEngine engine;
    engine.SetMemoryLimit(256 * 1024 * 1024);
    engine.SetTimeout(5000);
    
    uce::ExecutionResult result = engine.ExecutePython(task.script);
    
    if (result.status != uce::ExecutionStatus::Success) {
        // Log failure
        std::cerr << "Task failed: " << task.name << std::endl;
        
        // Optionally retry with different settings
        std::cerr << "Retrying with increased limits..." << std::endl;
        
        engine.SetMemoryLimit(512 * 1024 * 1024);
        engine.SetTimeout(10000);
        
        result = engine.ExecutePython(task.script);
        
        if (result.status != uce::ExecutionStatus::Success) {
            std::cerr << "Retry also failed. Skipping task." << std::endl;
        }
    }
}
```

## Important Notes

- ✅ Each `CommandEngine` instance = separate child process
- ✅ Child processes are **guaranteed** to be cleaned up
- ✅ No orphan processes remain after execution
- ✅ Shared parent process coordinates all children
- ❌ Do NOT reuse engine across multiple scripts (safety best practice)
- ❌ Do NOT assume state persists between executions

## Performance Characteristics

- **Process creation overhead**: 50-200 ms per task (first run)
- **Context switching**: <1 ms between parent and child
- **IPC communication**: <5 ms per message
- **Output capture**: Minimal overhead
- **Memory overhead**: ~5 MB parent + configured child minimums

## Next Steps

- See [sandbox_violation.md](sandbox_violation.md) for security examples
- See [memory_limit.md](memory_limit.md) for memory enforcement
- See [API.md](../API.md) for complete API reference
- See [EMBEDDING_GUIDE.md](../EMBEDDING_GUIDE.md) for integration patterns

---

UCE v0.1 Beta handles process isolation and cleanup automatically. Focus on your application logic!
