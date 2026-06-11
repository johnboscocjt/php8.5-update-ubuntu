# Updating to PHP 8.5 on Ubuntu

This guide walks through installing PHP 8.5 on Ubuntu using the **Ondřej Surý** PHP repository (`packages.sury.org`), resolving common extension dependency conflicts, and fixing **phpMyAdmin** errors caused by PCRE JIT restrictions.

Tested on **Ubuntu 26.04 LTS (Resolute)**. Steps also apply to Ubuntu 22.04/24.04 with minor codename changes.

---

## Prerequisites

- Ubuntu 20.04, 22.04, 24.04, or 26.04
- `sudo` privileges
- Active internet connection
- Existing web stack (Apache or Nginx) if you use phpMyAdmin or Laravel

---

## Step 1 — Add the PHP Repository

Ubuntu’s default PHP packages are often older and **will not match** Sury PHP 8.5/8.5.7 builds. Use the Sury repository directly.

### Option A: Sury repository (recommended)

Replace `resolute` with your Ubuntu codename if different (`noble` for 24.04, `jammy` for 22.04):

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# GPG key (skip if already present)
sudo curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg \
  https://packages.sury.org/php/apt.gpg

# Enable repo — use YOUR Ubuntu codename
echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ resolute main" \
  | sudo tee /etc/apt/sources.list.d/php.list

sudo apt update
```

### Option B: Ondřej Surý PPA (alternative)

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

> **Important:** Do not mix Ubuntu’s default `php8.5-*` packages (e.g. `8.5.4-0ubuntu1.1`) with Sury’s `php8.5-common` (e.g. `8.5.7-...`). That causes dependency conflicts like:
>
> ```
> php8.5-bcmath : Depends: php8.5-common (= 8.5.4-0ubuntu1.1) but 8.5.6-... is to be installed
> ```

---

## Step 2 — Install PHP 8.5 and Core Extensions

Install CLI and commonly required extensions **from the same repository**:

```bash
sudo apt install -y \
  php8.5-cli \
  php8.5-common \
  php8.5-mysql \
  php8.5-pgsql \
  php8.5-sqlite3 \
  php8.5-zip \
  php8.5-gd \
  php8.5-mbstring \
  php8.5-curl \
  php8.5-xml \
  php8.5-bcmath \
  php8.5-readline
```

### Quick fix script (JArchitect repo)

From the project root:

```bash
sudo bash developer/fix-php-extensions.sh
```

This script enables the Sury `resolute` repo and installs `php8.5-bcmath`, `php8.5-gd`, and `php8.5-sqlite3`.

---

## Step 3 — Install Server-Specific Packages

### For Nginx (PHP-FPM)

```bash
sudo apt install -y php8.5-fpm
```

Update your site config to use the PHP 8.5 socket:

```nginx
fastcgi_pass unix:/run/php/php8.5-fpm.sock;
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

### For Apache (mod_php)

```bash
sudo apt install -y libapache2-mod-php8.5
```

Switch Apache to PHP 8.5:

```bash
# Disable older PHP modules (adjust versions to match your system)
sudo a2dismod php8.3 php8.4
sudo a2enmod php8.5

sudo systemctl restart apache2
```

> If you keep multiple PHP modules enabled, Apache may still serve an older version. Check with:
>
> ```bash
> ls /etc/apache2/mods-enabled/php*
> ```

---

## Step 4 — Set PHP 8.5 as the CLI Default

When multiple PHP versions are installed:

```bash
sudo update-alternatives --set php /usr/bin/php8.5
php -v
```

Expected output (version may differ):

```
PHP 8.5.7 (cli) ...
```

---

## Step 5 — Verify Extensions

```bash
php -m | grep -E 'bcmath|gd|sqlite|pdo_sqlite|curl|mbstring|xml|zip'
```

You should see at minimum:

```
bcmath
curl
gd
mbstring
pdo_sqlite
sqlite3
xml
zip
```

---

## Step 6 — Fix phpMyAdmin (PCRE JIT Error)

### Symptom

phpMyAdmin shows:

```
Warning: preg_match(): Allocation of JIT memory failed, PCRE JIT will be disabled...
Error during session start...
session_start(): Session cannot be started after headers have already been sent
```

The warning prints output **before** HTTP headers, which breaks sessions.

### Cause

Security restrictions block PCRE JIT executable memory allocation under Apache’s PHP.

### Fix

Disable PCRE JIT for **Apache’s PHP** (not only CLI). Adjust PHP version to match what Apache uses (`8.3`, `8.4`, or `8.5`):

```bash
# For PHP 8.3 (common when php8.3 is still the active Apache module)
echo "pcre.jit=0" | sudo tee /etc/php/8.3/apache2/conf.d/99-disable-pcre-jit.ini

# For PHP 8.4 (if enabled)
echo "pcre.jit=0" | sudo tee /etc/php/8.4/apache2/conf.d/99-disable-pcre-jit.ini

# For PHP 8.5 (after switching Apache to 8.5)
echo "pcre.jit=0" | sudo tee /etc/php/8.5/apache2/conf.d/99-disable-pcre-jit.ini

sudo systemctl restart apache2
```

### Verify phpMyAdmin

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/phpmyadmin/
```

Expected: `200` with no orange/red warning blocks in the browser.

---

## Step 7 — Laravel / SQLite Setup (JArchitect Backend)

With `php8.5-sqlite3` installed, run migrations:

```bash
cd backend
php artisan migrate
```

Ensure `backend/.env` uses SQLite for local development:

```env
DB_CONNECTION=sqlite
```

---

## Troubleshooting

### Extension install fails with version conflict

**Problem:** `php8.5-bcmath` wants `php8.5-common (= 8.5.4-0ubuntu1.1)` but you have Sury `8.5.6+`.

**Fix:**

1. Ensure Sury repo is enabled (`/etc/apt/sources.list.d/php.list`).
2. Use the correct codename (`resolute`, `noble`, etc.).
3. Run `sudo apt update` then reinstall extensions from Sury only.

### phpMyAdmin still broken after PCRE fix

1. Confirm which PHP module Apache uses: `ls /etc/apache2/mods-enabled/php*`
2. Apply `pcre.jit=0` to **that** version’s `apache2/conf.d/` directory.
3. Restart Apache: `sudo systemctl restart apache2`
4. Hard-refresh the browser (Ctrl+Shift+R).

### Disabled Sury repo

If you find `php.list.disabled`, re-enable it:

```bash
sudo mv /etc/apt/sources.list.d/php.list.disabled /etc/apt/sources.list.d/php.list
# Update codename inside the file if needed (noble → resolute on Ubuntu 26.04)
sudo apt update
```

---

## Optional Cleanup

After confirming all apps work on PHP 8.5:

```bash
sudo apt purge php8.3* php8.4*   # adjust versions
sudo apt autoremove
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Check CLI version | `php -v` |
| List loaded modules | `php -m` |
| Apache PHP modules | `ls /etc/apache2/mods-enabled/php*` |
| Restart Apache | `sudo systemctl restart apache2` |
| Test phpMyAdmin | `curl -I http://localhost/phpmyadmin/` |
| Laravel migrate | `cd backend && php artisan migrate` |

---

## Additional Notes

- The Ondřej Surý repository is widely trusted and maintained by a Debian developer.
- On Ubuntu 26.04, use the **`resolute`** suite at `packages.sury.org/php`.
- Always install PHP and its extensions from the **same source** (all Sury or all Ubuntu — never mixed).
- Test applications in a staging environment before upgrading production systems.
- If issues persist, check `/var/log/apache2/error.log` and `php -i` for `Loaded Configuration File`.

---

## Related Links

- [Ondřej Surý PPA on Launchpad](https://launchpad.net/~ondrej/+archive/ubuntu/php)
- [Sury PHP packages](https://packages.sury.org/php/)
- [JArchitect stack overview](./stack.md)

---

**Last updated:** June 2026 — Ubuntu 26.04 LTS, PHP 8.5.7, phpMyAdmin 5.2
