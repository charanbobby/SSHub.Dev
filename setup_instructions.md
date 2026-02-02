# Hetzner Cloud VPS Setup

## Quick Access (Login & Updates)

**Login:**
```powershell
ssh -i "$env:USERPROFILE\.ssh\id_hetzner" sri@[SERVER_IP]
```

**Maintenance & Updates:**
See the dedicated [Maintenance Guide](file:///e:/!Python%20Applications/Hetzner%20Cloud%20Setup/maintenance_guide.md) for full details.

**Quick System Update:**
```bash
sudo apt update && sudo apt upgrade -y
```

## 1. SSH Key Generation (Custom Identity)

It is good practice to use a separate SSH key for your cloud servers.

Open **PowerShell** or **Command Prompt** and run the following command:

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com" -f "$env:USERPROFILE\.ssh\id_hetzner"
```

1. **File Path**: The `-f` flag specifies the output file (e.g., `id_hetzner`).
2. **Passphrase**: You can enter a secure passphrase or leave it empty for password-less login.

This will generate:

* `id_hetzner`: Your **PRIVATE** key.
* `id_hetzner.pub`: Your **PUBLIC** key.

### Option B: Rename Existing Key

If you already generated a key (e.g., `id_ed25519`) and want to use it for Hetzner, you can rename it:

```powershell
Move-Item "$env:USERPROFILE\.ssh\id_ed25519" "$env:USERPROFILE\.ssh\id_hetzner"
Move-Item "$env:USERPROFILE\.ssh\id_ed25519.pub" "$env:USERPROFILE\.ssh\id_hetzner.pub"
```

### Copy Public Key

To copy your new public key to the clipboard:

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_hetzner.pub" | Set-Clipboard
```

## 2. Connecting to the Server

Connect to your server (IP: `[SERVER_IP]`) using the `root` user and your `id_hetzner` key.

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_hetzner" root@[SERVER_IP]
```
```powershell
ssh -i "E:\!Python Applications\SSH\id_hetzner" root@[SERVER_IP]
```

*Note: The command above works from any directory. If you are already in your `.ssh` folder, you can also use `-i .\id_hetzner`.*

## 3. Initial Server Setup (Run explicitly on Server)

After logging in as **root**, run the following commands to create a new user and transfer "ownership" (admin privileges and SSH access).

### Step 1: Create New User & Grant Sudo Access

```bash
# 1. Create the new user "sri"
# When prompted for a password, enter: admin
adduser sri

# 2. Add "sri" to the sudo group (gives admin rights)
usermod -aG sudo sri
```

### Step 2: Transfer SSH Access

Copy the authorized SSH keys from `root` to `sri` so you can log in with the same key.

```bash
# 1. Copy .ssh directory with correct permissions
rsync --archive --chown=sri:sri ~/.ssh /home/sri

# 2. Ensure permissions are secure
chmod 700 /home/sri/.ssh
chmod 644 /home/sri/.ssh/authorized_keys
```

### Step 3: Verify & secure

**Do not close your current root session yet.** Open a **new** terminal on your computer and try to log in as `sri`:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_hetzner" sri@[SERVER_IP]
```

If the login works, you can (optional but recommended) disable root login for security.

**Back in your server session:**

```bash
# Edit the SSH config file
nano /etc/ssh/sshd_config
```

Find the line `PermitRootLogin yes` and change it to:
`PermitRootLogin no`

Save: `Ctrl+O`, `Enter`
Exit: `Ctrl+X`

Finally, restart SSH to apply changes:
```bash
systemctl restart ssh
```

## 4. Web Server & Firewall Setup

To host a website, you need to install a web server (Nginx) and **configure the firewall** to allow traffic.

### Step 1: Install Nginx

```bash
sudo apt install nginx -y
```

### Step 2: Configure Firewall (UFW) - CRITICAL

By default, the firewall might block web traffic. You must explicitly allow SSH (so you don't lock yourself out) and Nginx.

```bash
# 1. Allow SSH connections (IMPORTANT!)
sudo ufw allow OpenSSH

# 2. Allow Nginx HTTP/HTTPS traffic
sudo ufw allow 'Nginx Full'

# 3. Enable the firewall
sudo ufw enable
# Type 'y' and press Enter if prompted.
```

### Step 3: Verify

Check the status of Nginx and the Firewall:

```bash
# Check Nginx status
sudo systemctl status nginx

# Check Firewall status
sudo ufw status
```

Now, try visiting your IP address (`http://[SERVER_IP]`) in your browser. You should see the "Welcome to nginx!" page.

## 5. Docker Installation

Run the following commands to install Docker Engine and Docker Compose.

### Step 1: Clean up old versions
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### Step 2: Setup Repository & Key
```bash
# 1. Install dependencies
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y

# 2. Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 3. Add the repository to Apt sources
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 3: Install Docker
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### Step 4: Run Docker without Sudo
Add your user `sri` to the docker group so you don't need `sudo` for every docker command.

```bash
sudo usermod -aG docker sri
```

**Important:** You must **log out and log back in** for this change to effect.
```bash
exit
# Then ssh back in
```
