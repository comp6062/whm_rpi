# Web Hosting with Webmin on Raspberry Pi (AArch64)

This guide will walk you through setting up a web hosting environment on a Raspberry Pi (AArch64 architecture) using Webmin. Additionally, we will cover setting up custom name servers to direct your domains to the Raspberry Pi.

## Prerequisites

- Raspberry Pi 3, 4, or 5 (AArch64 architecture)
- Raspberry Pi OS installed
- SSH access to your Raspberry Pi
- A purchased domain name

## Step 1: Install Webmin

### 1. Install Required Dependencies

First, update your system packages and install the required dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common apt-transport-https wget -y
```

### 2. Add the Webmin Repository

Add the Webmin repository and GPG key to your system:

```bash
wget -qO - http://www.webmin.com/jcameron-key.asc | sudo tee /etc/apt/trusted.gpg.d/webmin.asc
sudo add-apt-repository "deb https://download.webmin.com/download/repository sarge contrib"
```

### 3. Install Webmin

Now, update your package list and install Webmin:

```bash
sudo apt update && sudo apt install webmin -y
```

### 4. Open Webmin's Port

Webmin runs on port `10000` by default. Open this port in your firewall:

```bash
sudo ufw allow 10000/tcp
sudo ufw enable
```

### 5. Access Webmin

Access Webmin by visiting:

```
https://<your-raspberry-pi-ip>:10000
```

Log in using your Raspberry Pi’s root username and password.

## Step 2: Install Apache, PHP, and MariaDB

### 1. Install Apache, PHP, and MariaDB

```bash
sudo apt install apache2 php php-mysql mariadb-server -y
```

### 2. Enable and Start Services

Ensure the web and database services are enabled and running:

```bash
sudo systemctl enable --now apache2 mariadb
```

### 3. Secure MariaDB

Run the security script to set up a root password and remove insecure defaults:

```bash
sudo mysql_secure_installation
```

Follow the prompts to improve security:
- Set a **root password**.
- Remove **anonymous users**.
- Disallow **remote root login**.
- Remove **test databases**.
- Reload privileges.

### 4. Verify Installation

Check if MariaDB is running:

```bash
sudo systemctl status mariadb
```

You can also log in to MariaDB:

```bash
sudo mysql -u root -p
```

Enter your root password to access the database.

## Step 3: Set Up Virtual Hosts for Your Domains

### 1. Add a New Virtual Server in Webmin

- In the Webmin dashboard, navigate to `Servers > Apache Webserver > Create a new virtual host`.
- Enter your domain name, document root (e.g., `/var/www/html/example.com`), and other necessary fields.

### 2. Configure DNS Records

To point your domain to your Raspberry Pi’s IP address, log in to your domain registrar (e.g., GoDaddy, Namecheap) and create the following DNS records:

- **A Record**: Points `example.com` to your Raspberry Pi’s external IP address.
- **CNAME Record**: If you need subdomains (e.g., `www.example.com`), set up CNAME records pointing to the root domain.

If you have a dynamic IP address, consider using a service like [DuckDNS](https://www.duckdns.org/) to keep your IP updated.

## Step 4: Set Up Custom Name Servers

To create custom name servers (`ns1.example.com` and `ns2.example.com`), follow these steps:

### 1. Install Bind9 (DNS Server)

```bash
sudo apt install bind9 bind9utils bind9-doc -y
```

### 2. Configure DNS Zones

Edit the Bind9 configuration file:

```bash
sudo nano /etc/bind/named.conf.local
```

Add this configuration:

```bash
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};
```

Now, create the zone file:

```bash
sudo nano /etc/bind/db.example.com
```

Add the following:

```bash
$TTL 86400
@    IN    SOA   ns1.example.com. admin.example.com. (
                2025040201 ; Serial
                28800      ; Refresh
                7200       ; Retry
                1209600    ; Expire
                86400 )    ; Minimum TTL
;
@    IN    NS    ns1.example.com.
@    IN    NS    ns2.example.com.
;
@    IN    A     <your-raspberry-pi-ip>
ns1  IN    A     <your-raspberry-pi-ip>
ns2  IN    A     <your-raspberry-pi-ip>
www  IN    CNAME example.com.
```

Replace `<your-raspberry-pi-ip>` with your Raspberry Pi’s actual IP address.

### 3. Restart Bind9

```bash
sudo systemctl restart bind9
```

### 4. Configure Your Domain Registrar

Log in to your domain registrar and set your custom name servers (`ns1.example.com` and `ns2.example.com`) for your domain.

## Step 5: Test Your Setup

### 1. Test Domain Resolution

After DNS propagation (which may take a few hours), test if your domain resolves correctly:

```bash
nslookup example.com
```

It should return your Raspberry Pi’s IP address.

### 2. Test the Website

Visit `http://example.com` in a browser to check if your website is live.

## Summary

- **Webmin** simplifies managing your Raspberry Pi server, including Apache, MariaDB, and DNS.
- **Bind9** enables setting up custom name servers for your domain.
- Correctly configuring both DNS records at your registrar and on your Raspberry Pi ensures proper domain resolution.

By following these steps, you’ll have a fully functioning web hosting setup with custom name servers running on your Raspberry Pi.

