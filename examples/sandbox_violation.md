# Sandbox Violation Example

This example demonstrates the UCE sandbox security model by attempting various operations that are intentionally blocked for safety.

## Scenario

Show how the sandbox prevents dangerous operations while allowing safe Python functionality.

## Code

```cpp
#include <iostream>
#include <vector>
#include <uce/CommandEngine.h>

struct TestCase {
    std::string name;
    std::string script;
    bool should_fail;  // true = expect sandbox violation
};

int main() {
    std::cout << "=== UCE Sandbox Violation Examples ===" << std::endl;
    std::cout << std::endl;
    std::cout << "This demonstrates UCE's security sandbox." << std::endl;
    std::cout << std::endl;
    
    // Test cases: mix of allowed and blocked operations
    std::vector<TestCase> tests = {
        // ALLOWED operations
        {
            "Safe: Print statement",
            "print('Hello, this is allowed')",
            false
        },
        {
            "Safe: Math operations",
            "result = 10 + 5 * 2\nprint(f'Result: {result}')",
            false
        },
        {
            "Safe: List operations",
            "items = [1, 2, 3]\nprint(f'Items: {items}')",
            false
        },
        {
            "Safe: String manipulation",
            "text = 'Hello'.upper()\nprint(f'Upper: {text}')",
            false
        },
        
        // BLOCKED operations
        {
            "Blocked: os module import",
            "import os\nprint(os.getcwd())",
            true
        },
        {
            "Blocked: sys module import",
            "import sys\nprint(sys.version)",
            true
        },
        {
            "Blocked: subprocess module",
            "import subprocess\nsubprocess.call(['cmd'])",
            true
        },
        {
            "Blocked: exec() function",
            "exec('print(\"dynamic code\")')",
            true
        },
        {
            "Blocked: eval() function",
            "result = eval('2 + 2')\nprint(result)",
            true
        },
        {
            "Blocked: getattr trick",
            "__import__('os').system('dir')",
            true
        },
    };
    
    int passed = 0;
    int failed = 0;
    
    for (const auto& test : tests) {
        std::cout << "Test: " << test.name << std::endl;
        std::cout << "Expected: ";
        std::cout << (test.should_fail ? "BLOCKED" : "ALLOWED") << std::endl;
        
        uce::CommandEngine engine;
        engine.SetMemoryLimit(128 * 1024 * 1024);
        engine.SetTimeout(5000);
        
        uce::ExecutionResult result = engine.ExecutePython(test.script);
        
        bool is_blocked = (result.status == uce::ExecutionStatus::SandboxViolation);
        bool test_passed = (is_blocked == test.should_fail);
        
        std::cout << "Result: ";
        
        if (is_blocked) {
            std::cout << "BLOCKED ✓" << std::endl;
        } else if (result.status == uce::ExecutionStatus::Success) {
            std::cout << "ALLOWED ✓" << std::endl;
            if (!result.stdout_output.empty()) {
                std::cout << "Output: " << result.stdout_output;
            }
        } else {
            std::cout << "ERROR: " << result.error_message << std::endl;
        }
        
        std::cout << "Test: ";
        if (test_passed) {
            std::cout << "PASS ✓" << std::endl;
            passed++;
        } else {
            std::cout << "FAIL ✗" << std::endl;
            failed++;
        }
        
        std::cout << std::endl;
    }
    
    std::cout << "=== Summary ===" << std::endl;
    std::cout << "Passed: " << passed << std::endl;
    std::cout << "Failed: " << failed << std::endl;
    std::cout << std::endl;
    
    if (failed == 0) {
        std::cout << "✓ All sandbox tests passed!" << std::endl;
        std::cout << "Sandbox is working as expected." << std::endl;
    } else {
        std::cout << "✗ Some tests failed." << std::endl;
        std::cout << "Security model may be compromised." << std::endl;
    }
    
    return failed == 0 ? 0 : 1;
}
```

## Running

```cmd
sandbox_violation.exe
```

Expected output:

```
=== UCE Sandbox Violation Examples ===

This demonstrates UCE's security sandbox.

Test: Safe: Print statement
Expected: ALLOWED
Result: ALLOWED ✓
Output: Hello, this is allowed
Test: PASS ✓

Test: Safe: Math operations
Expected: ALLOWED
Result: ALLOWED ✓
Output: Result: 30
Test: PASS ✓

Test: Safe: List operations
Expected: ALLOWED
Result: ALLOWED ✓
Output: Items: [1, 2, 3]
Test: PASS ✓

Test: Blocked: os module import
Expected: BLOCKED
Result: BLOCKED ✓
Test: PASS ✓

Test: Blocked: sys module import
Expected: BLOCKED
Result: BLOCKED ✓
Test: PASS ✓

Test: Blocked: exec() function
Expected: BLOCKED
Result: BLOCKED ✓
Test: PASS ✓

=== Summary ===
Passed: 10
Failed: 0

✓ All sandbox tests passed!
Sandbox is working as expected.
```

## What's Allowed

✅ **Safe Python features**:
- Print, string, list, dictionary operations
- Math and built-in functions
- Exception handling (try/except)
- Loops and conditionals
- Function definitions (locally scoped)
- Safe modules: `json`, `math`, `random`, `datetime`, `collections`

## What's Blocked

❌ **Dangerous operations**:
- **os module**: No OS-level file/process access
- **sys module**: No system introspection
- **subprocess module**: No process spawning
- **importlib module**: No bypass of import restrictions
- **exec() function**: No dynamic code execution
- **eval() function**: No expression evaluation
- **getattr() magic**: No attribute tricks
- **__import__()**: No dynamic imports

## Sandbox Limitations

**Important**: The sandbox prevents **accidental** misuse and **cooperative** script errors.

It does **NOT protect against**:
- Exploits of Python interpreter itself
- OS kernel vulnerabilities
- Determined attackers with security knowledge
- Side-channel attacks

For truly adversarial code, use OS-level isolation (containers, VMs).

## Handling Sandbox Violations

```cpp
uce::ExecutionResult result = engine.ExecutePython(user_script);

if (result.status == uce::ExecutionStatus::SandboxViolation) {
    std::cerr << "Sandbox violation: " << result.error_message << std::endl;
    std::cerr << "The script attempted to use blocked functionality." << std::endl;
    
    // Decide how to respond:
    // - Log and reject
    // - Ask user to modify script
    // - Provide error feedback
    
    return;  // Skip execution
}

if (result.status == uce::ExecutionStatus::Success) {
    std::cout << "Script executed successfully" << std::endl;
    std::cout << result.stdout_output << std::endl;
}
```

## Testing Your Scripts

Before deploying user scripts, test them:

```cpp
// Step 1: Create test instance
uce::CommandEngine test_engine;
test_engine.SetMemoryLimit(128 * 1024 * 1024);
test_engine.SetTimeout(5000);

// Step 2: Execute user's script
uce::ExecutionResult test_result = test_engine.ExecutePython(user_script);

// Step 3: Check for sandbox violations
if (test_result.status == uce::ExecutionStatus::SandboxViolation) {
    std::cerr << "Script rejected: " << test_result.error_message << std::endl;
    std::cerr << "Please modify your script to avoid blocked operations." << std::endl;
    return false;  // Reject script
}

// Step 4: If test passes, script is safe to deploy
std::cout << "Script validation passed." << std::endl;
return true;
```

## Common Blocked Patterns

If your script is being blocked, look for these patterns:

```python
# ❌ Blocked
import os                  # ← Remove this import
import sys                 # ← Remove this import
import subprocess          # ← Remove this import

exec("code")              # ← Use regular Python instead
eval("expression")        # ← Use regular Python instead

# ❌ Blocked (alternative syntax)
__import__('os')          # ← Blocked
importlib.import_module() # ← Blocked
getattr(obj, 'attr')      # ← Use obj.attr instead

# ✅ Allowed
print("output")
x = 10 + 20
data = [1, 2, 3]
result = max(data)
```

## Next Steps

- See [memory_limit.md](memory_limit.md) for memory constraint examples
- See [API.md](../API.md) for complete sandbox documentation
- See [SECURITY.md](../SECURITY.md) for security model details
- See [EMBEDDING_GUIDE.md](../EMBEDDING_GUIDE.md) for integration patterns

---

The UCE sandbox is designed for **cooperative scripts**, not adversarial code. Use OS-level isolation (containers, VMs) for untrusted code.
