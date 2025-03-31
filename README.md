**Raspberry Pi 5 (aarch64) - Installing WHM Manager on Debian Bullseye-based OS**

### Prerequisites

* Raspberry Pi 5 with a Debian Bullseye-based OS installed (aarch64/arm64 architecture)
* Internet connection
* A static IP address or a dynamic DNS service (recommended)

### Step 1: Update the package list and upgrade the system

Update the package list:
```
sudo apt update -y
```

Upgrade the system:
```
sudo apt full-upgrade -y
```

### Step 2: Install required packages

Install the following packages:
```
sudo apt install build-essential git libssl-dev libpam0g-dev libapr1-dev libaprutil1-dev libmysqlclient21-dev mysql-server perl perl-modules zip -y

```

### Step 3: Install Apache

Install Apache:
```
sudo apt install -y apache2
```

### Step 4: Install MySQL

Install MySQL:
```
sudo apt install -y mysql-server
```

### Step 5: Configure MySQL

Configure MySQL:
```
sudo mysql_secure_installation
```

Follow the prompts to set a root password, remove anonymous users, disallow root login remotely, and reload privilege tables.

### Step 6: Install WHM Manager

Install WHM Manager:
```
sudo apt install -y whm-manager
```

### Step 7: Configure WHM Manager

Configure WHM Manager:
```
sudo whm-manager --config /etc/whm-manager.conf
```

Follow the prompts to set the administrator password and configure other settings.

### Step 8: Start and enable services

Start and enable Apache, MySQL, and WHM Manager:
```
sudo systemctl start apache2
sudo systemctl enable apache2

sudo systemctl start mysql
sudo systemctl enable mysql

sudo systemctl start whm-manager
sudo systemctl enable whm-manager
```

### Step 9: Configure firewall (optional)

Configure the firewall to allow incoming connections on port 2087 (WHM Manager):
```
sudo ufw allow 2087
```

### Step 10: Access WHM Manager

Access WHM Manager by visiting `http://your-server-ip:2087` in your web browser.

That's it! You should now have a fully functional personal web server running WHM Manager on your Raspberry Pi 5.

