# System Dashboard

A comprehensive system monitoring dashboard built with [Python scripts](python.md) for real-time system metrics collection and visualization. Verify dashboard metrics using [Linux system commands](linux.md).

## Features

### Real-time Monitoring
- **CPU Usage** - Real-time CPU percentage and load averages
- **Memory Stats** - RAM usage, swap, and memory pressure indicators  
- **Disk Usage** - Storage utilization and I/O statistics
- **System Logs** - Automated logging to file for historical analysis

### Web Interface
- **Flask Application** - Extensible web interface for remote monitoring
- **API Endpoints** - RESTful API for programmatic access to metrics
- **Real-time Updates** - Live data refresh and visualization

## Implementation

### Core Monitoring Script

```python
import psutil
import time
import json
from datetime import datetime

def collect_system_stats():
    """Collect real-time system statistics"""
    stats = {
        'timestamp': datetime.now().isoformat(),
        'cpu_percent': psutil.cpu_percent(interval=1),
        'memory_percent': psutil.virtual_memory().percent,
        'disk_percent': psutil.disk_usage('/').percent
    }
    return stats

# Logs to file for later review
def log_to_file(stats, filename='system_logs.json'):
    """Log system stats to JSON file"""
    with open(filename, 'a') as f:
        f.write(json.dumps(stats) + '\n')
```

### Flask Web Interface

```python
# Flask web interface extension
from flask import Flask, render_template, jsonify

app = Flask(__name__)

@app.route('/')
def dashboard():
    """Main dashboard page"""
    stats = collect_system_stats()
    return render_template('dashboard.html', stats=stats)

@app.route('/api/stats')
def api_stats():
    """API endpoint for real-time stats"""
    return jsonify(collect_system_stats())

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

## Integration

This dashboard integrates with:
- **[Python Projects](python.md)** - Core monitoring scripts and automation tools
- **[Linux Commands](linux.md)** - System verification and troubleshooting commands
- **[Networking](networking.md)** - Network monitoring and connectivity checks

