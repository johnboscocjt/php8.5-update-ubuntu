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

### 3. Install Server-Specific Packages

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

## ⚙️ Post-Installation Configuration

### Set PHP 8.5 as the CLI Default

If you have multiple PHP versions installed, update the command-line version. This determines which PHP version runs when you execute `php` commands in the terminal:

```bash
sudo update-alternatives --set php /usr/bin/php8.5
```

### Configure Your Web Server

#### For Nginx Users

Update your site configuration file (e.g., `/etc/nginx/sites-available/default`) to point to the new PHP-FPM socket:

```nginx
fastcgi_pass unix:/run/php/php8.5-fpm.sock;
```

Restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

#### For Apache Users

Disable the previous PHP module and enable PHP 8.5:

```bash
sudo a2dismod php8.4   # Replace 8.4 with your previous version
sudo a2enmod php8.5
sudo systemctl restart apache2
```

---

## ✅ Verify the Installation

Confirm that PHP 8.5 is now active:

```bash
php -v
```

Expected output:

```
PHP 8.5.x (cli) (built: ...)
```

You can also verify web server integration by creating a `phpinfo()` file in your web root and accessing it through a browser.

---

## 🧹 Clean Up Old Version (Optional)

Once you've confirmed that all applications are working correctly with PHP 8.5, you can remove the previous PHP version:

```bash
sudo apt purge php8.4*   # Replace 8.4 with your previous version
sudo apt autoremove
```

This frees up disk space and removes unnecessary packages.

---

## 📝 Additional Notes

- The Ondřej Surý PPA is widely trusted and maintained by a Debian developer
- Always test your applications in a staging environment before upgrading production systems
- Some extensions may have different names or availability depending on your Ubuntu version
- If you encounter issues, check the PHP-FPM or Apache error logs for troubleshooting

---

**Need help?** Visit the [Ondřej Surý PPA page](https://launchpad.net/~ondrej/+archive/ubuntu/php) for more information.
