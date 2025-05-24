# POP Testnet Node Setup Guide in Octa Space
### To paste the command on the web terminal, right-click and select paste
### Also, split the screen, right-click, a menu will appear, hit h or select split horizontally.
### Then pick the top terminal to work with first
---

## Step 1: Verify Python and Port 443 Access

```bash
apt update
```

```bash
apt install -y python3
```

```bash
python3 -m http.server 443
```
- Copy your node IP for your Octa Sessions, click on the running session and you'll see the IP, copy it.
- Visit: [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/)
- Paste your IP to check if port 443 is open.
- If open, return to the terminal, press `Ctrl+C`, and continue to step 2.
- If not, try another machine and repeat step one till you find one that's open to port 443.

---

## Step 2: Install Required Packages

```bash
apt update
```

```bash
apt install -y curl
```

```bash
apt install -y libssl-dev
```

```bash
apt install -y ca-certificates
```

```bash
apt install -y git
```

```bash
apt install -y nano
```

```bash
apt install -y iproute2
```

```bash
apt install -y tar
```

```bash
mkdir -p /opt/popcache
```

```bash
mkdir -p /opt/popcache/logs
```

---

## Step 3: Get and Configure POP Node

Generate a GitHub token if needed for private access. 

Here's how to do it:

Log in to GitHub.

Go to Settings > Developer Settings > Personal Access Tokens.

Generate a classic token and set a far date for it to expire you'll use in place for a password. 
Note: Store this token somewhere safe as GitHub won't show it to you again.

```bash
git clone https://github.com/fae713/pop.git
```
It will ask you for your GitHub Username and Password (Token)

```bash
cd pop
```

```bash
mv "pop-v0.3.0-linux-x64 (1).tar.gz" /opt/popcache
```
```bash
cd /
```
```bash
cd /opt/popcache
```

```bash
tar -xzf pop-v0.3.0-linux-*.tar.gz
```

```bash
chmod +x /opt/popcache/pop
```

```bash
nano config.json
```

Paste and edit the following JSON:

```json
{
  "pop_name": "your-pop-name",
  "pop_location": "Your Location, Country",
  "invite_code": "Enter your Invite Code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 0
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-node-name",
    "name": "Your Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your_discord_username",
    "telegram": "your_telegram_handle",
    "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS_FOR_REWARDS"
  }
}
```

Save: `Ctrl+O`, `Enter`, then `Ctrl+X`

Note, to get your pop location, run the script on the terminal below
```bash
realpath --relative-to=/usr/share/zoneinfo /etc/localtime
```

---

## Step 4: Increase File Descriptors

```bash
bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```

---

## Step 5: Create systemd Service

```bash
sudo bash -c 'cat > /etc/systemd/system/popcache.service << EOL
[Unit]
Description=POP Cache Node
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/popcache
ExecStart=/opt/popcache/pop
Restart=always
RestartSec=5
LimitNOFILE=65535
StandardOutput=append:/opt/popcache/logs/stdout.log
StandardError=append:/opt/popcache/logs/stderr.log
Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]
WantedBy=multi-user.target
EOL'
```

---

## Step 6: Create Startup Script

```bash
nano start-popcache.sh
```

Paste:

```bash
#!/bin/bash

export POP_CONFIG_PATH=/opt/popcache/config.json
mkdir -p /opt/popcache/logs

while true; do
    /opt/popcache/pop >> /opt/popcache/logs/stdout.log 2>> /opt/popcache/logs/stderr.log
    echo "Restarting popcache in 5s..."
    sleep 5
done
```

```bash
chmod +x /start-popcache.sh
```

```bash
nohup /start-popcache.sh &
```

---

## Step 7: Verify It's Running
On the second Terminal run the codes below
```bash
ss -tuln | grep -E ':80|:443'
```

Expected output:

```
LISTEN  0  128  0.0.0.0:80   0.0.0.0:*
LISTEN  0  128  0.0.0.0:443  0.0.0.0:*
```

---

## Step 8: Monitor Node

```bash
curl -sk https://localhost/state && echo -e "\n"
```

```bash
curl -sk https://localhost/metrics && echo -e "\n"
```

```bash
curl -sk https://localhost/health && echo -e "\n"
```
To get your node ID, run the code below
```bash
cat .pop_state.json
```
To see live metrics, Visit https://dashboard.testnet.pipe.network/node/ID](https://dashboard.testnet.pipe.network/node/ID)
---
Your node needs to run for 24 hours before it starts receiving data from the pipe network.

**Note**: Ensure to top up your Octa balance before your session ends, or you'll have to do this process again when you top up. So monitor it often!
