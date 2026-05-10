# UCE v0.1 Beta Installation Guide

## System Requirements

### Minimum

- **OS**: Windows 7 SP1 or later
- **RAM**: 2 GB
- **Disk**: 50 MB for installation
- **Python**: 3.9 or later (for Python script execution)

### Recommended

- **OS**: Windows 10 or later
- **RAM**: 4 GB or more
- **Disk**: 100 MB available (for binaries and examples)
- **Python**: 3.10 or 3.11 (latest stable)

## Download

1. Go to: github.com/Sanji1-Owner/Universal-Command-Engine/releases
2. Find **UCE v0.1 Beta**
3. Download the appropriate package for your platform:
   - `uce-v0.1-beta-windows-x64.zip` (64-bit, recommended)
   - `uce-v0.1-beta-windows-x86.zip` (32-bit, legacy)

## Installation Steps

### Step 1: Extract Archive

1. Download the release ZIP file
2. Right-click → **Extract All**
3. Choose extraction location (e.g., `C:\Program Files\UCE` or `C:\opt\uce`)

```
C:\opt\uce\
├── bin/
│   ├── uce_command_engine.dll
│   └── uce_script_runner.exe
├── lib/
│   └── uce_command_engine.lib
├── include/
│   └── uce/
│       ├── CommandEngine.h
│       ├── ScriptRunner.h
│       └── ExecutionResult.h
├── examples/
├── docs/
└── README.md
```

### Step 2: Add to System PATH (Optional)

For convenient access from command line:

**Windows 10/11:**
1. Right-click **This PC** → **Properties**
2. Click **Advanced system settings**
3. Click **Environment Variables**
4. Under **System variables**, click **Path** → **Edit**
5. Click **New**
6. Add: `C:\opt\uce\bin`
7. Click **OK** → **OK** → **OK**

**Command line alternative:**
```powershell
$path = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$path;C:\opt\uce\bin", "User")
```

### Step 3: Verify Installation

1. Open **Command Prompt** or **PowerShell**
2. Test if DLL is accessible:

```cmd
dir C:\opt\uce\bin\
```

Expected output:
```
 Volume in drive C: is ...
 Directory of C:\opt\uce\bin

03/15/2024  10:30 AM    <DIR>          .
03/15/2024  10:30 AM    <DIR>          ..
03/15/2024  10:30 AM         2,340,000 uce_command_engine.dll
03/15/2024  10:30 AM           485,000 uce_script_runner.exe
```

3. Test Python integration (if Python 3.9+ installed):

```cmd
python --version
```

Expected: `Python 3.9.x` or later

## Integration with Your Project

### For C++ Projects

See [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md) for detailed integration steps.

Quick summary:
1. Include headers: `#include <uce/CommandEngine.h>`
2. Link library: `uce_command_engine.lib`
3. Copy DLLs to output directory
4. Call API from your code

### For Other Languages (v1.0+)

Currently v0.1 provides C++ API only.

Planned for v1.0:
- C# bindings
- Python wrapper
- Command-line interface

## Troubleshooting

### DLL Not Found

```
Error: The program can't start because uce_command_engine.dll is missing
```

**Solution:**
- Verify DLL exists in `bin/` folder
- Copy DLLs to your application directory
- Or add `C:\opt\uce\bin` to system PATH

### Python Not Found

```
Error: Python 3.9 or later not found
```

**Solution:**
1. Download Python from python.org
2. Install with "Add Python to PATH" checked
3. Verify: `python --version`
4. If still not found, add Python bin directory to PATH

### IPC Connection Error

```
Error: Failed to connect to child process (IPC error)
```

**Solution:**
- Ensure all UCE DLLs are from same v0.1 release
- Check Windows User Account Control (UAC) is not blocking
- Verify `uce_script_runner.exe` exists in `bin/`

### Permission Denied

```
Error: Permission denied writing to working directory
```

**Solution:**
- Run application as Administrator
- Or install UCE to user-writable location (e.g., user home directory)
- Check disk space availability

### Memory Allocation Failed

```
Error: Cannot allocate 256 MB (insufficient memory)
```

**Solution:**
- Close other applications to free memory
- Reduce memory limit in your code
- Check available RAM: `systeminfo | find "Total Physical Memory"`

## Configuration

### Environment Variables (Optional)

Create a `.env` file (or use system environment variables):

```env
UCE_HOME=C:\opt\uce
UCE_LOG_LEVEL=INFO
UCE_SANDBOX_LEVEL=strict
```

**Note**: Not required for basic usage. Defaults are sensible.

## Uninstallation

1. Delete installation directory: `C:\opt\uce`
2. Remove from PATH (if added):
   - Edit Environment Variables
   - Remove `C:\opt\uce\bin` from Path

## Next Steps

- Read [README.md](README.md) for overview
- See [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md) to integrate into your project
- Check [examples/](examples/) for code samples
- Review [API.md](API.md) for complete API reference
- See [docs/FAQ.md](docs/FAQ.md) for common questions

## Support

For installation issues:
- Check [SECURITY.md](SECURITY.md) for security-related questions
- See [docs/FAQ.md](docs/FAQ.md) for frequently asked questions
- File an issue: github.com/Sanji1-Owner/Universal-Command-Engine/issues

## Beta Notice

This is **UCE v0.1 Beta** — pre-release software for evaluation and testing.

- Not recommended for production deployment
- API may change before v1.0 (June 24, 2026)
- Third-party audit pending for v1.0
- Report bugs to support team

---

Installation complete! Now integrate UCE into your project using [EMBEDDING_GUIDE.md](EMBEDDING_GUIDE.md).
