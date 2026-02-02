# Moltbot Factory Reset Guide

This guide will walk you through completely removing the existing Moltbot installation and data from your server, and then reinstalling it from scratch (factory defaults).

> [!WARNING]
> **Data Loss Warning**: This process will delete ALL Moltbot data, including your pairing links, conversation history (if stored locally), and configurations. You will need to re-pair your Telegram account.

## Prerequisites

*   SSH access to your server (`sshub.dev`).
*   Sudo privileges.

## Step 1: Remove Existing Installation

1.  **SSH into your server**:
    ```powershell
    ssh -i "d:\!Python Applications\Hetzner Cloud Setup\id_hetzner_temp" sri@[SERVER_IP]
    ```
    *(Adjust the key path if you are running this from a different location, or just use your standard SSH command)*

2.  **Navigate to the Moltbot directory**:
    ```bash
    cd /opt/moltbot
    ```

3.  **Stop and Remove Containers & Volumes**:
    This command stops the containers and removes the associated Docker volumes (where data is stored).
    ```bash
    sudo docker compose down -v
    ```

4.  **Remove the Directory**:
    Go up one level and delete the entire moltbot folder.
    ```bash
    cd ..
    sudo rm -rf moltbot
    ```

## Step 2: Reinstall Moltbot (Factory Default)

1.  **Clone the Repository**:
    ```bash
    cd /opt
    sudo git clone https://github.com/moltbot/moltbot.git
    sudo chown -R $USER:$USER moltbot
    cd moltbot
    ```

2.  **Run Setup Script**:
    ```bash
    ./docker-setup.sh
    ```
    *Follow the prompts in the script.*

## Step 3: Re-configure Telegram

Since we wiped the configuration, you need to re-add your Telegram Bot Token.

1.  **Edit the .env file**:
    ```bash
    nano .env
    ```
    Add your token:
    ```bash
    TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
    ```

2.  **Restart Moltbot**:
    ```bash
    sudo docker compose restart
    ```

3.  **Pairing**:
    Message your bot on Telegram. It will give you a code.
    Run on server:
    ```bash
    sudo docker compose run --rm moltbot-cli pairing approve telegram <PAIRING_CODE>
    ```

## Verification

Check if Moltbot is running:
```bash
sudo docker compose ps
```
You should see the services `up`.
