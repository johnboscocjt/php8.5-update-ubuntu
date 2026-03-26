# Updating to PHP 8.5 on Ubuntu

This guide provides the necessary commands to upgrade your Ubuntu system to PHP 8.5 using the Ondřej Surý PPA.

---

## 📋 Prerequisites

- Ubuntu 20.04, 22.04, or 24.04
- Sudo privileges
- An active internet connection

---

## 🚀 Installation Steps

### 1. Add the PHP Repository

Ubuntu's default repositories often lag behind. Add the most trusted PHP PPA:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

### 2. Install PHP 8.5 and Core Extensions

Install the base PHP CLI along with the most commonly used extensions:

```bash
sudo apt install -y php8.5-cli php8.5-common php8.5-mysql php8.5-zip php8.5-gd php8.5-mbstring php8.5-curl php8.5-xml php8.5-bcmath
```

### 3. Server-Specific Installation

Depending on your web server stack, run one of the following:

**For Nginx (FPM)**

```bash
sudo apt install -y php8.5-fpm
```

**For Apache (Mod-PHP)**

```bash
sudo apt install -y libapache2-mod-php8.5
```

---

## ⚙️ Configuration

### Set PHP 8.5 as the System Default

If you have multiple versions installed, use `update-alternatives` to set the default CLI version:

```bash
sudo update-alternatives --set php /usr/bin/php8.5
```

### Update Web Server Links

#### Nginx

Update your site configuration file (e.g., `/etc/nginx/sites-available/default`):

```nginx
# Change this line
fastcgi_pass unix:/run/php/php8.5-fpm.sock;
```

Then restart Nginx:

```bash
sudo systemctl restart nginx
```

#### Apache

Disable the old module and enable the new one:

```bash
sudo a2dismod php8.4   # Change 8.4 to your previous version
sudo a2enmod php8.5
sudo systemctl restart apache2
```

---

## ✅ Verification

Confirm that the installation was successful:

```bash
php -v
```

Output should resemble:

```
PHP 8.5.x (cli) (built: ...)
```

---

## 🧹 Cleanup (Optional)

Once you have verified your applications are running correctly, you can remove the old version:

```bash
sudo apt purge php8.4*
sudo apt autoremove
```
