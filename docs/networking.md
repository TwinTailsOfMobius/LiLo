# Networking

This section covers networking concepts, tools, and commands that complement the [Linux system administration](linux.md) and [Python monitoring projects](python.md).

## Network Commands

```bash
# Check network connectivity
ping google.com

# Display network interface information
ip addr show
# or
ifconfig

# Show network connections and listening ports
netstat -tuln
# or
ss -tuln

# Display routing table
ip route show
# or
route -n

# Test DNS resolution
nslookup google.com
# or
dig google.com
```

## Network Monitoring

```bash
# Monitor network traffic in real-time
iftop
# or
nethogs

# Display network statistics
netstat -i
# or
ip -s link show

# Monitor bandwidth usage
vnstat
```

## Firewall and Security

```bash
# Check firewall status (UFW)
sudo ufw status

# Check iptables rules
sudo iptables -L

# Test port connectivity
telnet hostname port
# or
nc -zv hostname port
```

## Network Troubleshooting

```bash
# Trace network path
traceroute google.com
# or
tracepath google.com

# Check network configuration
ip link show
ip addr show
ip route show

# Test network speed
speedtest-cli
```

## Integration with System Monitoring

These networking commands can be integrated with the [System Dashboard](dashboard.md) to provide comprehensive network monitoring alongside CPU, memory, and disk metrics.
