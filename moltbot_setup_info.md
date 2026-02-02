# Moltbot (OpenClaw) Command Cheat Sheet

This document serves as a comprehensive reference for managing your Moltbot installation on the Hetzner server.

## 🔑 Key Configuration

| Item | Value |
| :--- | :--- |
| **Status** | Up |
| **Config Path** | `/home/sri/.openclaw` |
| **Workspace Path** | `/home/sri/.openclaw/workspace` |
| **Gateway Token** | `6b5b0de2d9e684580ad0d71a79b2e9e95e21ce8b5045c26a769f86ed62237c34` |
| **Docker Compose** | `/opt/moltbot/docker-compose.yml` |

---

## ⚙️ Configuration Management

The "Health Offline" status usually means the agent is starting up or needs configuration.

**Edit Configuration (Safely):**
```bash
# Edit the main config file
nano /home/sri/.openclaw/openclaw.json

# Restart to apply changes
sudo docker compose -f /opt/moltbot/docker-compose.yml restart
```

*Common Config Edits: Enabling keys, changing model settings, or adding custom tools.*

---

## 🚀 Service Management

Run these commands from `/opt/moltbot` or use the full path.

**Check Status:**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml ps
```

**Start Services:**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml up -d
```

**Stop Services:**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml down
```

**Restart Services:**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml restart
```

---

## 📊 Logs & Monitoring

**View Gateway Logs (Live):**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml logs -f openclaw-gateway
```

**Run Health Check:**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml exec openclaw-gateway node dist/index.js health --token "6b5b0de2d9e684580ad0d71a79b2e9e95e21ce8b5045c26a769f86ed62237c34"
```

---

## 🔌 Provider Setup & Pairing

Use these commands to connect Moltbot to your messaging platforms.

**Approve Telegram Pairing Code:**
*Run this when the bot asks for approval on Telegram.*
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli pairing approve telegram <PAIRING_CODE>
```

**Add Telegram Bot (Token):**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli providers add --provider telegram --token <YOUR_BOT_TOKEN>
```

**Add Discord Bot (Token):**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli providers add --provider discord --token <YOUR_BOT_TOKEN>
```

**Login to WhatsApp (QR Code):**
```bash
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli providers login
```

---

## 🛠️ Troubleshooting

**Check Running Processes:**
```bash
# Clean view of PIDs and names
pgrep -a openclaw

# Full detailed view
ps aux | grep openclaw | grep -v grep
```

**Kill a Stuck Process:**
```bash
kill <PID>
# Force kill
kill -9 <PID>
```

**Docker Cleanup (Prune):**
*Removes unused containers, networks, and images (dangling).*
```bash
sudo docker system prune
```

---

## 🧨 Factory Reset (Danger Zone)

**Wipe All Data & Reinstall:**

1.  **Stop & Remove Volumes:**
    ```bash
    cd /opt/moltbot
    sudo docker compose down -v
    ```

2.  **Delete Directory:**
    ```bash
    cd ..
    sudo rm -rf moltbot
    ```

3.  **Re-clone & Setup:**
    ```bash
    sudo git clone https://github.com/moltbot/moltbot.git
    cd moltbot
    ./docker-setup.sh
    ```

---

## 🌐 Nginx Reverse Proxy (Optional)

If you want to expose the Moltbot UI at `moltbot.sshub.dev`.

**1. Create Config:**
```bash
sudo nano /etc/nginx/sites-available/moltbot.sshub.dev
```

**Content:**
```nginx
server {
    server_name moltbot.sshub.dev;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

**2. Enable & Secure:**
```bash
sudo ln -s /etc/nginx/sites-available/moltbot.sshub.dev /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d moltbot.sshub.dev
```

**3. Accessing the Dashboard:**
You MUST include your gateway token in the URL to authorize the connection:
`https://moltbot.sshub.dev/?token=YOUR_GATEWAY_TOKEN`

*Your current token is listed in the "Key Configuration" section above.*

## ⚠️ Troubleshooting: Token Mismatch

If `grep` returns nothing, the service might be crashing or not printing the token.

1.  **Restart the service:**
    ```bash
    sudo docker compose -f /opt/moltbot/docker-compose.yml restart openclaw-gateway
    ```

2.  **View FULL logs (No Grep):**
    ```bash
    sudo docker compose -f /opt/moltbot/docker-compose.yml logs openclaw-gateway
    ```

    *   **Success:** Look manually for a line starting with `Gateway token:`.
    *   **Failure:** Look for error messages (e.g., `Error: ...`, `Panic: ...`) at the bottom.

