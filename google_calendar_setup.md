# OpenClaw Google Calendar Skill Setup

This guide details how to add the Google Calendar integration to your existing OpenClaw (MoltBot) installation without re-running the full onboarding wizard.

> [!IMPORTANT]
> **Do not run `openclaw onboard`** if you wish to preserve your current configuration. Use the manual steps below to add the specific skill you need.

## 1. Access the OpenClaw Dashboard

You need to access the web interface of your OpenClaw instance.
- **URL**: Likely `http://<your-server-ip>:3000` or a configured domain (e.g., `https://moltbot.sshub.dev` if you set up a proxy).
- **Login**: If prompted, you might need your Gateway Token found in your setup info:
  - Token: `6b5b0de2d9e684580ad0d71a79b2e9e95e21ce8b5045c26a769f86ed62237c34`

## 2. Install the Google Calendar Skill

1.  **Navigate to Skills**: In the dashboard sidebar, click on **Skills** or the puzzle piece icon.
2.  **Search**: Type `calendar` or `google` in the search bar.
3.  **Select Skill**: Look for **"Google Calendar"** or sometimes labeled as **"gog (brew)"** in the registry.
4.  **Install**: Click the **Install** button.

## 3. Authenticate with Google

Once the skill is installed, you need to link your Google Account.

1.  Go to the **Integrations** or **Config** section of the newly installed skill.
2.  Click **Connect** or **Login with Google**.
3.  Follow the OAuth flow to authorize OpenClaw to access your calendar.
    *   *Note: You may see a "Unverified App" warning if using a personal project. You can proceed by clicking "Advanced" > "Go to ... (unsafe)".*

## 4. Verification

To verify the integration is working, you can try asking OpenClaw:
- "What is on my calendar for today?"
- "Schedule a meeting for tomorrow at 2 PM."

## Troubleshooting

If you cannot access the Dashboard, you can check installed skills via the CLI on your server:

```bash
# Check installed skills
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli skills list
```

If you need to install via CLI (advanced):
```bash
# Search for the exact package name
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli skills search calendar

# Install (replace <skill-name> with the actual name found, e.g., google-calendar)
sudo docker compose -f /opt/moltbot/docker-compose.yml run --rm openclaw-cli skills install <skill-name>
```
