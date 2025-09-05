# Bash Scripts

A collection of useful automation scripts for system administration, monitoring, and daily tasks. These scripts complement the [Linux commands](linux.md) and [Python monitoring tools](python.md) in this wiki.

## Safety Guidelines

### Before Running Any Script

1. **Read the script first** - Always review the code before execution
2. **Test in a safe environment** - Use a test system or VM when possible
3. **Backup important data** - Create backups before running system scripts
4. **Use dry-run mode** - Many scripts include `--dry-run` or `-n` options
5. **Check permissions** - Ensure you have appropriate permissions
6. **Monitor system resources** - Watch for high CPU/memory usage

### Script Execution Best Practices

```bash
# Make scripts executable
chmod +x script_name.sh

# Run with bash for better error handling
bash -x script_name.sh  # Debug mode
bash -n script_name.sh  # Syntax check only

# Log script output
./script_name.sh 2>&1 | tee script_output.log
```

## System Monitoring Scripts

### 1. System Health Check

```bash
#!/bin/bash
# system_health_check.sh - Comprehensive system health monitoring

set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo "=== System Health Check ==="
echo "Date: $(date)"
echo "Hostname: $(hostname)"
echo

# Check disk usage
echo -e "${YELLOW}Disk Usage:${NC}"
df -h | grep -E '^/dev/' | while read line; do
    usage=$(echo $line | awk '{print $5}' | sed 's/%//')
    if [ $usage -gt 90 ]; then
        echo -e "${RED}WARNING: $line${NC}"
    elif [ $usage -gt 80 ]; then
        echo -e "${YELLOW}CAUTION: $line${NC}"
    else
        echo -e "${GREEN}OK: $line${NC}"
    fi
done

# Check memory usage
echo -e "\n${YELLOW}Memory Usage:${NC}"
free -h

# Check load average
echo -e "\n${YELLOW}Load Average:${NC}"
uptime

# Check critical services
echo -e "\n${YELLOW}Critical Services:${NC}"
services=("sshd" "systemd-resolved" "NetworkManager")
for service in "${services[@]}"; do
    if systemctl is-active --quiet $service; then
        echo -e "${GREEN}✓ $service is running${NC}"
    else
        echo -e "${RED}✗ $service is not running${NC}"
    fi
done

echo -e "\n${GREEN}Health check completed!${NC}"
```

**How to run safely:**
```bash
# Make executable and run
chmod +x system_health_check.sh
./system_health_check.sh

# Run with logging
./system_health_check.sh | tee health_check_$(date +%Y%m%d).log
```

### 2. Log File Cleanup

```bash
#!/bin/bash
# log_cleanup.sh - Safe log file cleanup with rotation

set -euo pipefail

LOG_DIR="/var/log"
MAX_SIZE="100M"
KEEP_DAYS=30
DRY_RUN=false

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        --dry-run)
            DRY_RUN=true
            shift
            ;;
        --max-size)
            MAX_SIZE="$2"
            shift 2
            ;;
        --keep-days)
            KEEP_DAYS="$2"
            shift 2
            ;;
        *)
            echo "Usage: $0 [--dry-run] [--max-size SIZE] [--keep-days DAYS]"
            exit 1
            ;;
    esac
done

echo "Log cleanup starting..."
echo "Max size: $MAX_SIZE"
echo "Keep days: $KEEP_DAYS"
echo "Dry run: $DRY_RUN"
echo

# Function to safely rotate log file
rotate_log() {
    local logfile="$1"
    local size=$(stat -f%z "$logfile" 2>/dev/null || stat -c%s "$logfile" 2>/dev/null)
    local max_bytes=$(numfmt --from=iec "$MAX_SIZE")
    
    if [ $size -gt $max_bytes ]; then
        echo "Rotating $logfile (size: $(numfmt --to=iec $size))"
        if [ "$DRY_RUN" = false ]; then
            # Create backup with timestamp
            cp "$logfile" "${logfile}.$(date +%Y%m%d_%H%M%S).bak"
            # Truncate original file
            > "$logfile"
        fi
    fi
}

# Clean old log files
echo "Cleaning old log files..."
if [ "$DRY_RUN" = false ]; then
    find "$LOG_DIR" -name "*.log.*" -type f -mtime +$KEEP_DAYS -delete
    find "$LOG_DIR" -name "*.bak" -type f -mtime +$KEEP_DAYS -delete
else
    find "$LOG_DIR" -name "*.log.*" -type f -mtime +$KEEP_DAYS -print
    find "$LOG_DIR" -name "*.bak" -type f -mtime +$KEEP_DAYS -print
fi

echo "Log cleanup completed!"
```

**How to run safely:**
```bash
# Test with dry run first
chmod +x log_cleanup.sh
./log_cleanup.sh --dry-run

# Run with custom parameters
./log_cleanup.sh --max-size 50M --keep-days 14

# Run with logging
./log_cleanup.sh 2>&1 | tee cleanup_$(date +%Y%m%d).log
```

## File Management Scripts

### 3. Safe File Backup

```bash
#!/bin/bash
# safe_backup.sh - Create timestamped backups with verification

set -euo pipefail

BACKUP_DIR="/backup"
SOURCE_DIR="$1"
BACKUP_NAME="$(basename "$SOURCE_DIR")_$(date +%Y%m%d_%H%M%S)"

if [ $# -ne 1 ]; then
    echo "Usage: $0 <source_directory>"
    exit 1
fi

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: Source directory '$SOURCE_DIR' does not exist"
    exit 1
fi

# Create backup directory
mkdir -p "$BACKUP_DIR"

echo "Creating backup of $SOURCE_DIR..."
echo "Backup location: $BACKUP_DIR/$BACKUP_NAME"

# Create tar backup with compression
tar -czf "$BACKUP_DIR/$BACKUP_NAME.tar.gz" -C "$(dirname "$SOURCE_DIR")" "$(basename "$SOURCE_DIR")"

# Verify backup integrity
echo "Verifying backup..."
if tar -tzf "$BACKUP_DIR/$BACKUP_NAME.tar.gz" > /dev/null; then
    echo "✓ Backup verification successful"
    
    # Show backup size
    size=$(du -h "$BACKUP_DIR/$BACKUP_NAME.tar.gz" | cut -f1)
    echo "Backup size: $size"
    
    # Create checksum file
    sha256sum "$BACKUP_DIR/$BACKUP_NAME.tar.gz" > "$BACKUP_DIR/$BACKUP_NAME.sha256"
    echo "Checksum saved to: $BACKUP_DIR/$BACKUP_NAME.sha256"
else
    echo "✗ Backup verification failed"
    rm -f "$BACKUP_DIR/$BACKUP_NAME.tar.gz"
    exit 1
fi

echo "Backup completed successfully!"
```

**How to run safely:**
```bash
# Make executable
chmod +x safe_backup.sh

# Test with a small directory first
./safe_backup.sh /tmp/test_dir

# Backup important directories
./safe_backup.sh /home/user/documents
./safe_backup.sh /etc
```

### 4. Duplicate File Finder

```bash
#!/bin/bash
# find_duplicates.sh - Find and optionally remove duplicate files

set -euo pipefail

SEARCH_DIR="$1"
REMOVE_DUPLICATES=false
DRY_RUN=true

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        --remove)
            REMOVE_DUPLICATES=true
            DRY_RUN=false
            shift
            ;;
        --dry-run)
            DRY_RUN=true
            shift
            ;;
        *)
            SEARCH_DIR="$1"
            shift
            ;;
    esac
done

if [ ! -d "$SEARCH_DIR" ]; then
    echo "Usage: $0 [--remove] [--dry-run] <directory>"
    exit 1
fi

echo "Finding duplicates in: $SEARCH_DIR"
echo "Remove duplicates: $REMOVE_DUPLICATES"
echo "Dry run: $DRY_RUN"
echo

# Find files with same size and checksum
find "$SEARCH_DIR" -type f -exec md5sum {} + | sort | uniq -d -w 32 | while read checksum filename; do
    echo "Duplicate found: $filename"
    
    if [ "$REMOVE_DUPLICATES" = true ] && [ "$DRY_RUN" = false ]; then
        echo "  Removing duplicate..."
        rm "$filename"
    fi
done

echo "Duplicate search completed!"
```

**How to run safely:**
```bash
# Always start with dry run
chmod +x find_duplicates.sh
./find_duplicates.sh --dry-run /home/user/documents

# Remove duplicates (be very careful!)
./find_duplicates.sh --remove /tmp/test_files
```

## Automation Scripts

### 5. System Update Script

```bash
#!/bin/bash
# system_update.sh - Safe system update with logging and rollback

set -euo pipefail

LOG_FILE="/var/log/system_update_$(date +%Y%m%d_%H%M%S).log"
BACKUP_DIR="/backup/rpm_backup_$(date +%Y%m%d_%H%M%S)"

# Function to log messages
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to create package backup
create_backup() {
    log "Creating package backup..."
    mkdir -p "$BACKUP_DIR"
    
    # Backup current package list
    rpm -qa > "$BACKUP_DIR/installed_packages.txt"
    
    # Backup yum/dnf history
    if command -v dnf &> /dev/null; then
        dnf history > "$BACKUP_DIR/dnf_history.txt"
    elif command -v yum &> /dev/null; then
        yum history > "$BACKUP_DIR/yum_history.txt"
    fi
    
    log "Backup created in: $BACKUP_DIR"
}

# Function to perform update
perform_update() {
    log "Starting system update..."
    
    # Update package cache
    if command -v dnf &> /dev/null; then
        dnf makecache
        dnf update -y
    elif command -v yum &> /dev/null; then
        yum makecache
        yum update -y
    else
        log "ERROR: No supported package manager found"
        exit 1
    fi
    
    log "System update completed successfully"
}

# Function to verify system
verify_system() {
    log "Verifying system after update..."
    
    # Check critical services
    services=("sshd" "systemd-resolved" "NetworkManager")
    for service in "${services[@]}"; do
        if systemctl is-active --quiet $service; then
            log "✓ $service is running"
        else
            log "✗ $service is not running - may need attention"
        fi
    done
    
    # Check system load
    load=$(uptime | awk -F'load average:' '{print $2}')
    log "System load: $load"
}

# Main execution
main() {
    log "=== System Update Started ==="
    
    # Create backup before update
    create_backup
    
    # Perform update
    perform_update
    
    # Verify system
    verify_system
    
    log "=== System Update Completed ==="
    log "Log file: $LOG_FILE"
    log "Backup directory: $BACKUP_DIR"
}

# Run main function
main "$@"
```

**How to run safely:**
```bash
# Make executable (requires root)
sudo chmod +x system_update.sh

# Run with logging
sudo ./system_update.sh

# Check the log file
tail -f /var/log/system_update_*.log
```

### 6. Network Connectivity Monitor

```bash
#!/bin/bash
# network_monitor.sh - Monitor network connectivity and log issues

set -euo pipefail

LOG_FILE="/var/log/network_monitor.log"
CHECK_INTERVAL=60
HOSTS=("8.8.8.8" "1.1.1.1" "google.com")

# Function to log with timestamp
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to check connectivity
check_connectivity() {
    local host="$1"
    local result
    
    if ping -c 1 -W 5 "$host" &> /dev/null; then
        result="UP"
    else
        result="DOWN"
    fi
    
    echo "$result"
}

# Function to get network stats
get_network_stats() {
    echo "=== Network Statistics ==="
    ip addr show
    echo
    echo "=== Routing Table ==="
    ip route show
    echo
    echo "=== Active Connections ==="
    ss -tuln
}

# Main monitoring loop
monitor_network() {
    log "Starting network monitoring..."
    log "Monitoring hosts: ${HOSTS[*]}"
    log "Check interval: ${CHECK_INTERVAL} seconds"
    
    while true; do
        all_up=true
        
        for host in "${HOSTS[@]}"; do
            status=$(check_connectivity "$host")
            if [ "$status" = "DOWN" ]; then
                log "ALERT: $host is DOWN"
                all_up=false
            else
                log "OK: $host is UP"
            fi
        done
        
        if [ "$all_up" = false ]; then
            log "Network issues detected, collecting diagnostics..."
            get_network_stats >> "$LOG_FILE"
        fi
        
        sleep "$CHECK_INTERVAL"
    done
}

# Handle script termination
cleanup() {
    log "Network monitoring stopped"
    exit 0
}

trap cleanup SIGINT SIGTERM

# Start monitoring
monitor_network
```

**How to run safely:**
```bash
# Make executable
chmod +x network_monitor.sh

# Run in background
./network_monitor.sh &

# Monitor the log
tail -f /var/log/network_monitor.log

# Stop monitoring
pkill -f network_monitor.sh
```

## Script Management

### Creating Your Own Scripts

```bash
#!/bin/bash
# template.sh - Template for creating new scripts

set -euo pipefail  # Always include these safety options

# Script metadata
SCRIPT_NAME="$(basename "$0")"
VERSION="1.0"
AUTHOR="Your Name"

# Default values
DEFAULT_VALUE="default"

# Function to show usage
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] [ARGUMENTS]

Description of what this script does.

OPTIONS:
    -h, --help      Show this help message
    -v, --version   Show version information
    -d, --dry-run   Show what would be done without executing
    -q, --quiet     Suppress output

ARGUMENTS:
    input_file      Input file to process

EXAMPLES:
    $SCRIPT_NAME input.txt
    $SCRIPT_NAME --dry-run input.txt

EOF
}

# Function to log messages
log() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*"
}

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            usage
            exit 0
            ;;
        -v|--version)
            echo "$SCRIPT_NAME version $VERSION"
            exit 0
            ;;
        -d|--dry-run)
            DRY_RUN=true
            shift
            ;;
        -q|--quiet)
            QUIET=true
            shift
            ;;
        *)
            INPUT_FILE="$1"
            shift
            ;;
    esac
done

# Validate inputs
if [ -z "${INPUT_FILE:-}" ]; then
    echo "Error: Input file required"
    usage
    exit 1
fi

# Main script logic
main() {
    log "INFO" "Starting $SCRIPT_NAME"
    log "INFO" "Processing: $INPUT_FILE"
    
    # Your script logic here
    
    log "INFO" "Script completed successfully"
}

# Run main function
main "$@"
```

## Integration with Other Tools

These bash scripts work seamlessly with:

- **[Linux Commands](linux.md)** - Use these scripts to automate common Linux tasks
- **[Python Projects](python.md)** - Call Python scripts from bash for complex processing
- **[System Dashboard](dashboard.md)** - Use monitoring scripts to feed data to the dashboard
- **[Networking](networking.md)** - Network scripts complement networking commands

## Best Practices Summary

1. **Always use `set -euo pipefail`** for error handling
2. **Include help and usage information** in every script
3. **Add logging and timestamps** for debugging
4. **Test with dry-run mode** before actual execution
5. **Create backups** before making system changes
6. **Validate inputs** and handle edge cases
7. **Use meaningful variable names** and comments
8. **Follow the principle of least privilege** for permissions

Remember: **When in doubt, don't run the script!** Always review, test, and understand what a script does before executing it on important systems.
