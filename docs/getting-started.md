# Getting Started

Welcome to the system administration and monitoring wiki! This guide will help you set up your development environment with all the necessary tools to get started.

## Prerequisites

Before you begin, ensure you have:
- A Linux system (Ubuntu, CentOS, RHEL, or similar)
- Internet connection for downloading packages
- Basic familiarity with the command line

## Installation Guide

### 1. RHEL/CentOS Tools

For Red Hat Enterprise Linux or CentOS systems:

```bash
# Update system packages
sudo yum update -y

# Install essential development tools
sudo yum groupinstall -y "Development Tools"

# Install additional useful packages
sudo yum install -y vim git curl wget htop tree

# Install system monitoring tools
sudo yum install -y sysstat iotop nethogs

# For newer versions (RHEL 8+), use dnf instead of yum
sudo dnf update -y
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y vim git curl wget htop tree sysstat iotop nethogs
```

### 2. Python Installation

#### Option A: Using System Package Manager

```bash
# Install Python 3 and pip
sudo yum install -y python3 python3-pip

# Verify installation
python3 --version
pip3 --version

# Install development dependencies
sudo yum install -y python3-devel
```

#### Option B: Using pyenv (Recommended for Development)

```bash
# Install pyenv dependencies
sudo yum install -y git gcc zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel

# Install pyenv
curl https://pyenv.run | bash

# Add to shell profile
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

# Reload shell
source ~/.bashrc

# Install and use Python 3.11
pyenv install 3.11.0
pyenv global 3.11.0
```

### 3. Python Dependencies

Install the Python packages needed for system monitoring:

```bash
# Install system monitoring libraries
pip3 install psutil flask requests

# Install development tools
pip3 install black flake8 pytest

# Create requirements.txt for the project
cat > requirements.txt << EOF
psutil>=5.9.0
flask>=2.3.0
requests>=2.31.0
black>=23.0.0
flake8>=6.0.0
pytest>=7.4.0
EOF

# Install from requirements file
pip3 install -r requirements.txt
```

### 4. MkDocs Installation

Install MkDocs and the Material theme for the wiki:

```bash
# Install MkDocs and Material theme
pip3 install mkdocs mkdocs-material

# Install additional MkDocs plugins
pip3 install mkdocs-mermaid2-plugin mkdocs-git-revision-date-localized-plugin

# Verify installation
mkdocs --version
```

## Setting Up the Wiki

### 1. Clone or Download the Wiki

```bash
# If using git
git clone <your-repo-url> lilo-wiki
cd lilo-wiki

# Or create the directory structure manually
mkdir -p docs/assets
```

### 2. Verify Project Structure

Ensure your project has this structure:

```
lilo-wiki/
├── docs/
│   ├── assets/
│   ├── getting-started.md
│   ├── index.md
│   ├── linux.md
│   ├── python.md
│   ├── dashboard.md
│   └── networking.md
├── mkdocs.yml
└── requirements.txt
```

### 3. Test the Installation

```bash
# Test Python monitoring script
python3 -c "import psutil; print(f'CPU: {psutil.cpu_percent()}%')"

# Test MkDocs build
mkdocs build

# Check for any build errors
echo "Build completed successfully!"
```

## Serving the Wiki Locally

### 1. Start the Development Server

```bash
# Navigate to your wiki directory
cd lilo-wiki

# Start MkDocs development server
mkdocs serve

# The wiki will be available at: http://127.0.0.1:8000
```

### 2. Access the Wiki

Open your web browser and navigate to:
- **Local URL**: http://127.0.0.1:8000
- **Network URL**: http://your-server-ip:8000 (for remote access)

### 3. Development Workflow

```bash
# Start the server in the background
mkdocs serve --dev-addr=0.0.0.0:8000 &

# Edit your markdown files in docs/
# Changes will auto-reload in the browser

# Stop the server when done
pkill -f "mkdocs serve"
```

## Quick Start Commands

### System Monitoring

```bash
# Test system monitoring with Python
python3 -c "
import psutil
print(f'CPU: {psutil.cpu_percent()}%')
print(f'Memory: {psutil.virtual_memory().percent}%')
print(f'Disk: {psutil.disk_usage(\"/\").percent}%')
"
```

### Wiki Development

```bash
# Build static site
mkdocs build

# Serve locally
mkdocs serve

# Deploy to GitHub Pages (if configured)
mkdocs gh-deploy
```

## Troubleshooting

### Common Issues

#### Python Import Errors
```bash
# If psutil import fails
pip3 install --upgrade psutil

# If flask import fails
pip3 install --upgrade flask
```

#### MkDocs Build Errors
```bash
# Check MkDocs configuration
mkdocs build --verbose

# Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('mkdocs.yml'))"
```

#### Port Already in Use
```bash
# Find process using port 8000
sudo netstat -tulpn | grep :8000

# Kill the process
sudo kill -9 <PID>

# Or use a different port
mkdocs serve --dev-addr=127.0.0.1:8001
```

## Next Steps

Once you have everything installed:

1. **Explore the Wiki**: Start with [Linux Notes](linux.md) for essential commands
2. **Try Python Scripts**: Check out [Python Projects](python.md) for monitoring tools
3. **Set Up Dashboard**: Follow [System Dashboard](dashboard.md) for web interface setup
4. **Network Monitoring**: Learn [Networking](networking.md) commands and tools

## Additional Resources

- **MkDocs Documentation**: https://www.mkdocs.org/
- **Material Theme**: https://squidfunk.github.io/mkdocs-material/
- **Python psutil**: https://psutil.readthedocs.io/
- **Flask Documentation**: https://flask.palletsprojects.com/

Happy monitoring! 🚀
