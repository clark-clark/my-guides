

# Bahamut Node Installation and Usage Guide

This guide provides detailed instructions for installing, configuring, and managing a Bahamut node.

## Table of Contents

- [Advanced Installation and Configuration](#advanced-installation-and-configuration)
- [Node Management](#node-management)
- [Log Monitoring](#log-monitoring)
- [Automatic Restart Script](#automatic-restart-script)
- [API Usage Example](#api-usage-example)
- [Firewall Configuration Example](#firewall-configuration-example)

## Advanced Installation and Configuration

```bash
# Clone the repository
git clone https://github.com/fastexlabs/binaries.git
cd binaries

# Install dependencies (example for Ubuntu/Debian)
sudo apt update
sudo apt install -y build-essential curl

# Install Bahamut
chmod +x bahamut
./bahamut install

# Configure settings
nano Bahamut/sahara/sahara_config.toml

# Example of changing the port in the configuration
# Find and change the line:
# port = 30333
# to
# port = 30334
```

## Node Management

```bash
# Start the node
./bahamut start --network sahara

# Stop the node
./bahamut stop

# Check node status
./bahamut status

# Update the node
./bahamut update
```

## Log Monitoring

```bash
# View logs in real-time
tail -f Bahamut/sahara/logs/sahara.log

# Search for errors in logs
grep "ERROR" Bahamut/sahara/logs/sahara.log
```

## Automatic Restart Script

Create a file named `restart_bahamut.sh`:

```bash
#!/bin/bash

while true
do
    if ! pgrep -f "bahamut" > /dev/null
    then
        echo "Bahamut is not running. Restarting..."
        /path/to/bahamut start --network sahara
    else
        echo "Bahamut is running normally."
    fi
    sleep 300 # Check every 5 minutes
done
```

Make the script executable and run it:

```bash
chmod +x restart_bahamut.sh
./restart_bahamut.sh &
```

## API Usage Example

```python
import requests

# Assuming the API is available at localhost:8545
url = "http://localhost:8545"

# Example API request
payload = {
    "jsonrpc": "2.0",
    "method": "eth_blockNumber",
    "params": [],
    "id": 1
}

response = requests.post(url, json=payload)
print(response.json())
```

## Firewall Configuration Example

```bash
# Open necessary ports (replace 30334 with your port)
sudo ufw allow 30334/tcp
sudo ufw allow 30334/udp

# Apply changes
sudo ufw enable
```

These code examples should help you gain a deeper understanding of working with the Bahamut node. Remember that some details may vary depending on the specific version and configuration of Bahamut. Always refer to the official documentation for the most up-to-date information.

---

Feel free to contribute to this guide by submitting pull requests or opening issues for any corrections or improvements.
