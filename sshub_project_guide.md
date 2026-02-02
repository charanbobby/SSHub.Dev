# sshub.dev Project Guide

This guide details how to transform your Hetzner VPS (`sshub.dev`) into a versatile development and automation hub.

## 0. Prerequisites & Base Setup

Before starting specific projects, we need a robust foundation using **Docker** to keep services isolated and **Nginx** to manage subdomains.

### Step 1: Install Docker & Docker Compose
Most modern apps (like n8n) mimic their production environment best in Docker.

```bash
# 1. Remove old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# 2. Set up the repository
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 3. Add the repo
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# 5. Add your user 'sri' to the docker group (avoids using sudo for every docker command)
sudo usermod -aG docker sri
# (You must log out and back in for this to take effect)
```

### Step 2: Prepare Nginx for Subdomains
We will use Nginx as a "Reverse Proxy". Traffic comes to `n8n.sshub.dev` -> Nginx -> Internal Docker Container.

Ensure Nginx is running:
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## Project 1: n8n Automation Instance
**Goal:** Run n8n on `n8n.sshub.dev`.

### 1. DNS Setup
*   Go to your Domain Registrar (where you bought sshub.dev).
*   Add an **A Record**:
    *   Name: `n8n`
    *   Content: `[SERVER_IP]` (Your VPS IP)

### 2. Create Docker Compose File
We'll keep projects in `/opt`.

```bash
# Create directory
sudo mkdir -p /opt/n8n/data
sudo chown -R sri:sri /opt/n8n

# Create the compose file
nano /opt/n8n/docker-compose.yml
```

**Content for docker-compose.yml:**
```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678" # Only listen on localhost, Nginx will proxy to this
    environment:
      - N8N_HOST=n8n.sshub.dev
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://n8n.sshub.dev/
      - GENERIC_TIMEZONE=America/New_York # Adjust as needed
    volumes:
      - ./data:/home/node/.n8n
```

Start it:
```bash
cd /opt/n8n
docker compose up -d
```

### 3. Configure Nginx Proxy
```bash
sudo nano /etc/nginx/sites-available/n8n.sshub.dev
```

**Content:**
```nginx
server {
    server_name n8n.sshub.dev;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Required for n8n websockets
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/n8n.sshub.dev /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 4. Secure with SSL (Certbot)
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d n8n.sshub.dev
```

---

## Project 2: Shopify Development Redirect
**Goal:** `shopify.sshub.dev` redirects to a real store or password page.

### 1. DNS Setup
*   Add **A Record**: `shopify` -> `[SERVER_IP]`

### 2. Nginx Redirect Config
```bash
sudo nano /etc/nginx/sites-available/shopify.sshub.dev
```

**Content:**
```nginx
server {
    server_name shopify.sshub.dev;

    location / {
        # Redirect to your actual dev store URL
        return 301 https://your-store-name.myshopify.com;
    }
}
```

Enable & Secure:
```bash
sudo ln -s /etc/nginx/sites-available/shopify.sshub.dev /etc/nginx/sites-enabled/
sudo systemctl reload nginx
sudo certbot --nginx -d shopify.sshub.dev
```

---

## Project 3: Fullstack Projects (Infrastructure & Cyber)

### 1. Cybersecurity: Fail2Ban
Protect your SSH and Nginx from brute force.

```bash
sudo apt install fail2ban -y
```

Copy config to local overrides:
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```
*   Ensure `[sshd]` is `enabled = true`.
*   Establish basic bans (e.g., ban for 1 hour after 5 failed attempts).

Restart: `sudo systemctl restart fail2ban`

### 2. Fun Project: "Honeypot" Directory
Create a fake admin login page to see who scans your server.
*   Create `/var/www/html/admin/index.html` with a fake login form.
*   Monitor your Nginx access logs (`/var/log/nginx/access.log`) to see bots trying to POST data to it.
*   *Advanced:* Write a Python script to parse these logs and plot where the attacks are coming from (GeoIP).

---

## Project 4: Popular Self-Hosted Apps

Below are detailed, step-by-step instructions (DNS, Docker, Nginx) for each app.



### 1. Trilium Notes
**Goal:** Hierarchical note taking at `notes.sshub.dev`.

**Step 1: DNS Setup**
*   Add **A Record**: `notes` -> `[SERVER_IP]`

**Step 2: Docker Setup**
```bash
sudo mkdir -p /opt/trilium
sudo nano /opt/trilium/docker-compose.yml
```
**Content:**
```yaml
services:
  trilium:
    image: triliumnext/trilium:latest
    restart: unless-stopped
    environment:
      - TRILIUM_DATA_DIR=/home/node/trilium-data
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - ./trilium-data:/home/node/trilium-data
```
Run it:
```bash
cd /opt/trilium && docker compose up -d
```

**Step 3: Nginx Config**
```bash
sudo nano /etc/nginx/sites-available/notes.sshub.dev
```
**Content:**
```nginx
server {
    server_name notes.sshub.dev;
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```
**Step 4: Enable & SSL**
```bash
sudo ln -s /etc/nginx/sites-available/notes.sshub.dev /etc/nginx/sites-enabled/
sudo systemctl reload nginx
sudo certbot --nginx -d notes.sshub.dev
```

### 2. Uptime Kuma
**Goal:** Run monitoring at `status.sshub.dev`.

**Step 1: DNS Setup**
*   Add **A Record**: `status` -> `[SERVER_IP]`

**Step 2: Docker Setup**
```bash
sudo mkdir -p /opt/uptime-kuma
sudo nano /opt/uptime-kuma/docker-compose.yml
```
**Content:**
```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    restart: always
    ports:
      - "127.0.0.1:3001:3001"
    volumes:
      - ./uptime-kuma-data:/app/data
```
Run it:
```bash
cd /opt/uptime-kuma && docker compose up -d
```

**Step 3: Nginx Config**
```bash
sudo nano /etc/nginx/sites-available/status.sshub.dev
```
**Content:**
```nginx
server {
    server_name status.sshub.dev;
    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```
**Step 4: Enable & SSL**
```bash
sudo ln -s /etc/nginx/sites-available/status.sshub.dev /etc/nginx/sites-enabled/
sudo systemctl reload nginx
sudo certbot --nginx -d status.sshub.dev
```

### 3. Glances
**Goal:** System stats at `monitor.sshub.dev`.

**Step 1: DNS Setup**
*   Add **A Record**: `monitor` -> `[SERVER_IP]`

**Step 2: Docker Setup**
```bash
sudo mkdir -p /opt/glances
sudo nano /opt/glances/docker-compose.yml
```
**Content:**
```yaml
services:
  glances:
    image: nicolargo/glances:latest-full
    restart: always
    pid: host
    environment:
      - GLANCES_OPT=-w
    ports:
      - "127.0.0.1:61208:61208"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
```
Run it:
```bash
cd /opt/glances && docker compose up -d
```

**Step 3: Nginx Config**
```bash
sudo nano /etc/nginx/sites-available/monitor.sshub.dev
```
**Content:**
```nginx
server {
    server_name monitor.sshub.dev;
    location / {
        proxy_pass http://127.0.0.1:61208;
        proxy_set_header Host $host;
    }
}
```
**Step 4: Enable & SSL**
```bash
sudo ln -s /etc/nginx/sites-available/monitor.sshub.dev /etc/nginx/sites-enabled/
sudo systemctl reload nginx
sudo certbot --nginx -d monitor.sshub.dev
```

### 4. Nginx Proxy Manager (NPM)
**Goal:** UI Management at `npm.sshub.dev`.
**Note:** Since we are already using Nginx on ports 80/443, we will run NPM on **alternate ports** internally and proxy to its Admin UI. This keeps your current setup safe.

**Step 1: DNS Setup**
*   Add **A Record**: `npm` -> `[SERVER_IP]`

**Step 2: Docker Setup**
```bash
sudo mkdir -p /opt/npm
sudo nano /opt/npm/docker-compose.yml
```
**Content:**
```yaml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    restart: always
    ports:
      - "8081:81"   # Admin UI exposed on port 8081
      - "8090:80"   # HTTP traffic (alternate port - changed from 8080)
      - "8443:443"  # HTTPS traffic (alternate port)
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```
Run it:
```bash
cd /opt/npm && docker compose up -d
```

**Step 3: Nginx Config (Exposing Admin UI)**
```bash
sudo nano /etc/nginx/sites-available/npm.sshub.dev
```
**Content:**
```nginx
server {
    server_name npm.sshub.dev;
    location / {
        proxy_pass http://127.0.0.1:8081; # Pointing to the internal Admin Port
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```
**Step 4: Enable & SSL**
```bash
sudo ln -s /etc/nginx/sites-available/npm.sshub.dev /etc/nginx/sites-enabled/
sudo systemctl reload nginx
sudo certbot --nginx -d npm.sshub.dev
```
*Login with: `admin@example.com` / `changeme`*

### 5. Portainer
**Goal:** Docker Management at `portainer.sshub.dev`.

**Step 1: DNS Setup**
*   Add **A Record**: `portainer` -> `[SERVER_IP]`

**Step 2: Docker Setup**
```bash
sudo mkdir -p /opt/portainer
sudo nano /opt/portainer/docker-compose.yml
```
**Content:**
```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    restart: always
    ports:
      - "127.0.0.1:9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
volumes:
  portainer_data:
```
Run it:
```bash
cd /opt/portainer && docker compose up -d
```

**Step 3: Nginx Config**
```bash
sudo nano /etc/nginx/sites-available/portainer.sshub.dev
```
**Content:**
```nginx
server {
    server_name portainer.sshub.dev;
    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```
**Step 4: Enable & SSL**
```bash
sudo ln -s /etc/nginx/sites-available/portainer.sshub.dev /etc/nginx/sites-enabled/
sudo systemctl reload nginx
sudo certbot --nginx -d portainer.sshub.dev
```

---

### 6. Management Tip: Deploying via Portainer
You can install apps (like Glances or NPM) directly from your new Portainer dashboard instead of the terminal.

1.  **Login** to `portainer.sshub.dev`.
2.  Click **Stacks** (on the left) -> **+ Add stack**.
3.  **Name:** (e.g., `glances`).
4.  **Build method:** Web editor.
5.  **Web editor:** Paste the **Docker Compose content** (the YAML code) provided in the steps above.
6.  Click **Deploy the stack**.

*Note: You still need to do the **DNS Setup** and **Nginx Config** steps on the server, but this replaces the "Docker Setup" command-line steps.*

---

## Utility Redirects
Use Nginx to create handy shortlinks for your life/work.

**Example: `meet.sshub.dev` -> Zoom Personal Room**
Config: `/etc/nginx/sites-available/meet.sshub.dev`
```nginx
server {
    server_name meet.sshub.dev;
    location / {
        return 301 https://zoom.us/j/1234567890;
    }
}
```
*(Remember to enable it with `ln -s` and run Certbot)*

## Summary of URLs

| Service | Subdomain (Example) | Description |
| :--- | :--- | :--- |
| **n8n** | `n8n.sshub.dev` | Automation Workflows |
| **Trilium** | `notes.sshub.dev` | Knowledge Base |
| **Uptime Kuma** | `status.sshub.dev` | Uptime Monitoring |
| **Glances** | `monitor.sshub.dev` | System Stats |
| **Portainer** | `portainer.sshub.dev` | Docker Management |
| **Zoom Redirect** | `meet.sshub.dev` | Utility Shortlink |
| **Shopify** | `shopify.sshub.dev` | Redirect to Dev Store |

---

## Project 5: Moltbot (Telegram AI Assistant)
**Goal:** Run Moltbot as a self-hosted AI assistant with Telegram integration, isolated via Docker.

Moltbot will:
*   Listen to Telegram messages via a bot token.
*   Run fully containerized.
*   Not require public HTTP exposure unless you explicitly want the control UI.
*   Use pairing approval to prevent unauthorized access.

*(This fits the non-critical, experimentation-friendly nature of sshub.dev)*

### 1. Telegram Bot Setup (External)
1.  Open Telegram and start **@BotFather**.
2.  Run `/newbot`.
3.  Choose a name and username.
4.  Copy the **Bot Token** (keep it secret).

*Optional (for group chats):*
1.  Run `/setprivacy`.
2.  Disable privacy mode.

### 2. Docker Setup
We will follow the official Moltbot Docker flow and align it with your `/opt/<app>` convention.

```bash
cd /opt
sudo git clone https://github.com/moltbot/moltbot.git
sudo chown -R $USER:$USER moltbot
cd moltbot

# Run the official setup script (do NOT use sudo)
./docker-setup.sh
```

This will:
*   Build Moltbot images.
*   Generate a `.env` file.
*   Create a gateway token.
*   Start required services via Docker Compose.
*   Expose the Control UI **only on localhost** (`http://127.0.0.1:18789`).

### 3. Configure Telegram Channel
Edit the Moltbot config or `.env` file (created by the setup script):

```bash
sudo nano .env
```
Add:
```bash
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
```

Ensure Telegram is enabled in config (config file takes precedence over env vars if both exist):
```json
channels: {
  telegram: {
    enabled: true,
    dmPolicy: "pairing"
  }
}
```

Restart Moltbot:
```bash
sudo docker compose restart
```

### 4. Pairing & First Login (Security Step)
When you message the bot on Telegram for the first time:
1.  Moltbot will generate a pairing code.
2.  The bot will not respond until approved.

List pairing requests:
```bash
sudo docker compose run --rm moltbot-cli pairing list telegram
```

Approve your Telegram account:
```bash
sudo docker compose run --rm moltbot-cli pairing approve telegram <PAIRING_CODE>
```
*This ensures only approved users can interact with Moltbot.*

### 5. Optional: Expose Control UI via Nginx (Advanced)
**Recommended default:** Do not expose Moltbot publicly. Telegram does not require inbound webhooks.

If you **do** want UI access (for debugging), expose it securely.

**Step 1: DNS Setup**
*   Add **A Record**: `moltbot` -> `[SERVER_IP]`

**Step 2: Nginx Config**
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
    }
}
```

Enable and secure:
```bash
sudo ln -s /etc/nginx/sites-available/moltbot.sshub.dev /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d moltbot.sshub.dev
```
*Strongly recommended: protect this route with HTTP basic auth or restrict via VPN.*

### 6. Monitoring (Optional)
Add Moltbot to Uptime Kuma:
*   **Type:** HTTP
*   **URL:** `http://127.0.0.1:18789`
*   Expect status `200`.

---

## Summary of URLs

| Service | Subdomain (Example) | Description |
| :--- | :--- | :--- |
| **n8n** | `n8n.sshub.dev` | Automation Workflows |
| **Trilium** | `notes.sshub.dev` | Knowledge Base |
| **Uptime Kuma** | `status.sshub.dev` | Uptime Monitoring |
| **Glances** | `monitor.sshub.dev` | System Stats |
| **Portainer** | `portainer.sshub.dev` | Docker Management |
| **Moltbot** | *(Telegram)* | AI Assistant (Self-hosted) |
| **Moltbot UI** | `moltbot.sshub.dev` | Control UI (Optional) |
| **Zoom Redirect** | `meet.sshub.dev` | Utility Shortlink |
| **Shopify** | `shopify.sshub.dev` | Redirect to Dev Store |
