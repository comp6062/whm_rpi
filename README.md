

```markdown
# Web Hosting with Webmin on Raspberry Pi (aarch64)

This guide will walk you through setting up a web hosting environment on a Raspberry Pi (with aarch64 architecture) using Webmin. We'll also cover setting up custom name servers to direct your domains to the Raspberry Pi.

## Prerequisites

- Raspberry Pi 3 or 4 (with aarch64 architecture)
- Raspberry Pi OS installed
- SSH access to your Raspberry Pi
- A domain name (already purchased)

## Step 1: Install Webmin

### 1. Install required dependencies

First, update your system packages and install the required dependencies:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install software-properties-common apt-transport-https wget -y
```

### 2. Add the Webmin repository

Add the Webmin repository and GPG key to your system:

```bash
wget -qO - http://www.webmin.com/jcameron-key.asc | sudo tee /etc/apt/trusted.gpg.d/webmin.asc
sudo add-apt-repository "deb https://download.webmin.com/download/repository sarge contrib"
```

### 3. Install Webmin

Now, update your package list and install Webmin:

```bash
sudo apt update
sudo apt install webmin -y
```

### 4. Open Webmin's port

Webmin runs on port `10000` by default. Open this port in your firewall:

```bash
sudo ufw allow 10000/tcp
sudo ufw enable
```

### 5. Access Webmin

You can access Webmin by visiting:

```
https://<your-raspberry-pi-ip>:10000
```

Log in with the root username and password of your Raspberry Pi.

## Step 2: Install Apache, PHP, and MySQL

Webmin can manage Apache, MySQL, and PHP with ease. Install these services:

```bash
sudo apt install apache2 php php-mysql mysql-server -y
```

Ensure the services are running:

```bash
sudo systemctl enable apache2 mysql
sudo systemctl start apache2 mysql
```

## Step 3: Set Up Virtual Hosts for Your Domains

### 1. Add a new Virtual Server in Webmin

- In the Webmin dashboard, go to `Servers > Apache Webserver > Create a new virtual host`.
- Fill in the domain name, document root (e.g., `/var/www/html/example.com`), and other necessary fields.

### 2. Configure DNS Records

You need to point your domain to your Raspberry Pi’s IP address. Log in to your domain registrar (e.g., GoDaddy, Namecheap, etc.) and create the following DNS records:

- **A Record**: Point your domain (e.g., `example.com`) to your Raspberry Pi’s external IP address.
- **CNAME Record**: If you need subdomains (e.g., `www.example.com`), set up CNAME records to point them to the root domain.

If you're using a dynamic IP address, consider using a service like [DuckDNS](https://www.duckdns.org/) to keep your IP updated.

## Step 4: Set Up Custom Name Servers

To set up your own custom name servers (e.g., `ns1.example.com` and `ns2.example.com`), follow these steps:

### 1. Set Up Bind9 (DNS Server)

Install Bind9 to act as your DNS server:

```bash
sudo apt install bind9 bind9utils bind9-doc -y
```

### 2. Configure DNS Zones

The main configuration file for Bind9 is located at `/etc/bind/named.conf.local`. Create a new zone for your domain:

```bash
sudo nano /etc/bind/named.conf.local
```

Add the following configuration for your domain:

```bash
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};
```

Now, create the zone file `/etc/bind/db.example.com`:

```bash
sudo nano /etc/bind/db.example.com
```

Populate it with the following:

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

Replace `<your-raspberry-pi-ip>` with your Raspberry Pi’s IP address.

### 3. Restart Bind9

Restart the Bind9 service to apply changes:

```bash
sudo systemctl restart bind9
```

### 4. Configure Your Domain Registrar

Log in to your domain registrar and set the custom nameservers (`ns1.example.com` and `ns2.example.com`) for your domain. This will ensure that the domain resolves to your Raspberry Pi’s IP address.

## Step 5: Test Your Setup

### 1. Test the domain resolution

After DNS propagation (which may take a few hours), test if your domain is resolving correctly:

```bash
nslookup example.com
```

The IP should return your Raspberry Pi’s IP address.

### 2. Test the website

In a browser, visit `http://example.com` to see if your website is live.

## Summary

- **Webmin** is a user-friendly tool for managing your Raspberry Pi server, including Apache, MySQL, and DNS.
- **Bind9** can be configured to manage custom name servers for your domain.
- You’ll need to configure both DNS records at your registrar and on your Raspberry Pi to ensure that your domains resolve correctly.

By following these steps, you’ll have a fully functioning web hosting setup with custom name servers running on your Raspberry Pi.
```

### Key Points:
- Replace the `<your-raspberry-pi-ip>` placeholder with your actual Raspberry Pi IP address in the configuration files.
- The file paths and configuration details for Apache, Bind9, and Webmin are set up for common default installations; you may need to adjust them depending on your specific setup.
  
This Markdown format is compatible with GitHub, and you can copy it directly into a `.md` file for use on GitHub or a similar platform.
