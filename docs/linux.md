# Linux Notes

Essential Linux commands and shortcuts for system administration. These commands complement the [Python monitoring scripts](python.md) and [System Dashboard](dashboard.md).

## System Commands

```bash
# List all files with detailed information
ls -la

# Show disk usage in human-readable format
df -h

# Monitor processes (top or htop)
top
# or
htop
```

## Command Line Shortcuts

```bash
# Repeat the last command
!!

# Reverse search through command history
Ctrl+R
```

## Process Management

```bash
# Monitor processes (top or htop) - useful for verifying [Python dashboard](dashboard.md) metrics
top
# or
htop

# Kill processes by name
pkill process_name

# Find processes by name
pgrep process_name
```

