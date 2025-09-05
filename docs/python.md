# Python Projects

Python scripts and tools for system monitoring, automation, and development. These projects integrate with [Linux system commands](linux.md) and power the [System Dashboard](dashboard.md).

## System Monitoring

### Basic System Stats

```python
import psutil

# Monitor system CPU, memory, and disk
cpu_percent = psutil.cpu_percent(interval=1)
memory = psutil.virtual_memory()
disk = psutil.disk_usage('/')

print(f"CPU: {cpu_percent}%")
print(f"Memory: {memory.percent}%")
print(f"Disk: {disk.percent}%")
```

### Advanced Monitoring

```python
# Get detailed system information
import psutil
import datetime

def get_system_info():
    return {
        'timestamp': datetime.datetime.now().isoformat(),
        'cpu_count': psutil.cpu_count(),
        'cpu_percent': psutil.cpu_percent(interval=1),
        'memory': psutil.virtual_memory()._asdict(),
        'disk': psutil.disk_usage('/')._asdict(),
        'network': psutil.net_io_counters()._asdict()
    }
```

## Automation Tools

### Daily Reports

```python
# Automate daily system reports
import datetime
import json

def generate_daily_report():
    report = {
        'date': datetime.datetime.now().isoformat(),
        'cpu': psutil.cpu_percent(),
        'memory': psutil.virtual_memory().percent,
        'disk': psutil.disk_usage('/').percent
    }
    
    with open('daily_report.json', 'w') as f:
        json.dump(report, f, indent=2)
```

### CLI Tools

```python
# Build CLI mini-tools (complement [Linux commands](linux.md) for system administration)
import argparse

def main():
    parser = argparse.ArgumentParser(description='System Monitor CLI')
    parser.add_argument('--cpu', action='store_true', help='Show CPU usage')
    parser.add_argument('--memory', action='store_true', help='Show memory usage')
    parser.add_argument('--disk', action='store_true', help='Show disk usage')
    
    args = parser.parse_args()
    
    if args.cpu:
        print(f"CPU: {psutil.cpu_percent()}%")
    if args.memory:
        print(f"Memory: {psutil.virtual_memory().percent}%")
    if args.disk:
        print(f"Disk: {psutil.disk_usage('/').percent}%")

if __name__ == "__main__":
    main()
```

## Integration

These Python scripts can be integrated into the [System Dashboard](dashboard.md) for enhanced functionality and real-time monitoring capabilities.

