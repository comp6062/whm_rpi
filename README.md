# Raspberry Pi 5 Web Hosting Server Setup (AArch64/ARM64)

## Overview
This guide provides a step-by-step process to set up your Raspberry Pi 5 as a web hosting server with HestiaCP, complete with a web UI control panel, name server assignment, and domain management.

## Prerequisites
- Raspberry Pi 5
- MicroSD card (32GB+ recommended)
- USB Ethernet adapter (optional but recommended for dual network interfaces)
- AArch64/ARM64 Raspberry Pi OS (Debian Bullseye)
- Static public IP (recommended) or dynamic DNS (DDNS)
- Domain name(s) registered
- SSH access enabled

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

## Step 2: Install Required Software & Dependencies
### Install Web Server Stack
```bash
sudo apt install -y nginx php-fpm php-mysql mariadb-server unzip curl bind9 bind9-utils ufw
```

### Secure MariaDB
```bash
sudo mysql_secure_installation
```
Follow the prompts to secure the database and create a root password.

## Step 3: Install and Configure HestiaCP
### Install HestiaCP
```bash
curl -O https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
sudo bash hst-install.sh
```
Follow the prompts to configure the panel. Choose options suitable for your use case (Nginx + PHP-FPM recommended).

### Firewall Configuration
Enable UFW (Uncomplicated Firewall) and allow necessary ports:
```bash
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 8083/tcp # HestiaCP Web UI
sudo ufw enable
```

### Access HestiaCP Web UI
1. Open your browser and navigate to:
   ```
   https://<your_pi_ip>:8083
   ```
2. Log in with the default username `admin` and the password provided after installation.

### Add a New Domain in HestiaCP
1. Go to the **Web** section in the control panel.
2. Click **Add Domain** and enter your domain name.
3. Enable **SSL and Let's Encrypt** if required.
4. Click **Save** to apply the settings.

## Step 4: Configure Name Servers (Bind9)
### Install Bind9 and Setup DNS Zones
```bash
sudo apt install -y bind9 bind9utils
```

Edit Bind9 configuration:
```bash
sudo nano /etc/bind/named.conf.local
```
Add the following zone configuration (replace `yourdomain.com` with your actual domain):
```
zone "yourdomain.com" {
    type master;
    file "/etc/bind/db.yourdomain";
};
```

Create a DNS zone file:
```bash
sudo cp /etc/bind/db.local /etc/bind/db.yourdomain
sudo nano /etc/bind/db.yourdomain
```
Modify it as follows:
```
;
; BIND data file for yourdomain.com
;
$TTL    604800
@       IN      SOA     ns1.yourdomain.com. admin.yourdomain.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.yourdomain.com.
@       IN      NS      ns2.yourdomain.com.
ns1     IN      A       <your_pi_ip>
ns2     IN      A       <your_pi_ip>
@       IN      A       <your_pi_ip>
www     IN      A       <your_pi_ip>
```
Save and exit.

Restart Bind9:
```bash
sudo systemctl restart bind9
```

## Step 5: Enable SSL with Let's Encrypt
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

## Step 6: Verify Setup and Deployment
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
Your Raspberry Pi 5 is now a fully functional web hosting server with a control panel, DNS management, and SSL encryption. 🎉

