# Raspberry Pi 5 Web Hosting Server Setup (AArch64/ARM64)

This guide details how to set up your Raspberry Pi 5 as a web hosting server, complete with a web UI control panel and name server assignment.

## Prerequisites
- Raspberry Pi 5
- MicroSD card (32GB+ recommended)
- USB Ethernet adapter (optional but recommended for dual network interfaces)
- AArch64/ARM64 Raspberry Pi OS (Debian Bullseye)
- Static public IP (recommended) or dynamic DNS (DDNS)
- Domain name(s) registered

## Step 1: Install Raspberry Pi OS
1. Download the latest **Raspberry Pi OS (64-bit, Lite version)** from the official Raspberry Pi website.
2. Flash the OS to a MicroSD card using **Raspberry Pi Imager** or **Balena Etcher**.
3. After flashing, create an empty file named `ssh` in the boot partition to enable SSH.
4. Insert the MicroSD card into the Raspberry Pi and boot it up.
5. Connect via SSH:
   ```bash
   ssh pi@<raspberry_pi_ip>
   ```
6. Update the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

## Step 2: Install Required Software
### Install Web Server (Nginx + PHP + MariaDB)
```bash
sudo apt install -y nginx php-fpm php-mysql mariadb-server unzip curl
```

### Secure MariaDB
```bash
sudo mysql_secure_installation
```
Follow the prompts to secure the database.

## Step 3: Install a Web Hosting Control Panel
### Install HestiaCP (Recommended)
```bash
curl -O https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
sudo bash hst-install.sh
```
Follow the prompts to configure the panel.

## Step 4: Configure Name Servers
1. Register a domain and set up Glue records at your domain registrar.
2. Install and configure Bind9:
   ```bash
   sudo apt install -y bind9
   ```
3. Configure Bind9 to act as an authoritative name server for your domains.
4. Set up A and NS records in the Bind9 zone files.

## Step 5: Add a Website
1. Log into HestiaCP at `https://<your_pi_ip>:8083`
2. Create a new user (optional) and add a new domain.
3. Upload website files to `/home/user/web/domain.tld/public_html/`

## Step 6: Enable SSL with Let's Encrypt
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

## Step 7: Test & Deploy
- Ensure web traffic reaches your Raspberry Pi (port forwarding may be needed).
- Verify DNS propagation using:
  ```bash
  dig yourdomain.com @8.8.8.8
  ```
- Monitor logs with:
  ```bash
  sudo journalctl -u nginx -f
  ```

## Conclusion
Your Raspberry Pi 5 is now a fully functional web hosting server with a control panel and custom name servers. 🎉

