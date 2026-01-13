# sshub.dev Maintenance & Update Guide

This guide covers how to keep your VPS and its services up-to-date, secure, and running smoothly.

## 1. Operating System (OS) Updates

Run these monthly (or when prompted after login) to maintain security and stability.

```bash
# Update the list of available packages
sudo apt update

# Install all available updates
sudo apt upgrade -y

# Remove old, unused packages to save space
sudo apt autoremove -y
```

> [!NOTE]
> If a kernel update is installed, you may be prompted to restart. Use `sudo reboot` and wait 30 seconds before logging back in.

---

## 2. Docker Service Updates

To update apps like **n8n**, **Glances**, or **Portainer**, you need to "pull" the latest version and recreate the container.

### Fast Way (Individual App)
```bash
cd /opt/<app-name>
docker compose pull
docker compose up -d
```

### Pro-Tip: Update Everything at Once
Navigate to `/opt` and run this loop to update all projects:
```bash
for d in /opt/*/ ; do (cd "$d" && docker compose pull && docker compose up -d); done
```

---

## 3. SSL / Security Maintenance

### SSL Certificates (Certbot)
Certbot automatically renews certificates. You can verify they are working:
```bash
sudo certbot renew --dry-run
```

### Firewall (UFW)
Check your open ports occasionally:
```bash
sudo ufw status
```

### Brute Force Protection (Fail2Ban)
See how many IPs have been banned for trying to guess your passwords:
```bash
sudo fail2ban-client status sshd
```

---

## 4. Backups (Critical)

Data for your apps is stored in `/opt/<app-name>/data` (or similar volumes).

### Manual Backup (Simple)
To back up a specific app's data to your home directory:
```bash
# Example for n8n
sudo tar -czvf ~/n8n_backup_$(date +%F).tar.gz /opt/n8n/data
```

### Disaster Recovery
If the server crashes, you only need:
1. Your **Docker Compose files** (`/opt/*/docker-compose.yml`)
2. Your **Data volumes** (`/opt/*/data`)

> [!TIP]
> Periodically download these `.tar.gz` files to your local computer using WinSCP or `scp`.

---

## 5. System Health Check

Run these commands to see if your server is healthy:

- **Check Disk Space:** `df -h` (Ensure `/` is not at 100%)
- **Check Memory:** `free -h`
- **Monitor Real-time:** `glances` (or visit `https://monitor.sshub.dev`)
- **Container Status:** `docker ps`
