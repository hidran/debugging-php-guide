# Install PHP on Windows with Herd (Laravel) – Complete Guide

This page walks you through installing **PHP on Windows** using **Herd** (the installer recommended by the Laravel ecosystem). By the end, you’ll have PHP working, added to your **PATH**, **Composer** ready, and the ability to manage **multiple PHP versions** side by side.

---

## 1) Why Herd
**Herd** is a runtime manager that:
- installs **multiple PHP versions** (8.1/8.2/8.3/…);
- bundles common **extensions** (curl, mbstring, intl, openssl, etc.);
- provides a **launcher** (`herd php`, `herd composer`) and can set your **PATH**.

Compared to manual zip/DLL installs or WAMP/XAMPP stacks, Herd is more **maintainable**, works great with **Laravel**, and simplifies switching **PHP versions** per project.

---

## 2) Requirements
- Windows 10/11 (updated)
- Admin rights to install software
- Internet connection
- (Recommended) **PowerShell** terminal

---

## 3) Install Herd
1. Download the installer from **https://herd.laravel.com**.
2. Run the installer and follow the steps.
3. Open **Herd** and go to the **PHP** section.

> Tip: close terminals/IDEs during installation and reopen them at the end to inherit updated environment variables.

---

## 4) Install PHP with Herd
1. In **Herd → PHP**, choose a version (e.g., 8.3/8.2) and click **Install**.
2. Set the **Default** version.
3. Verify in terminal:
   - `php -v` → shows the expected version;
   - `herd php -v` → forces Herd’s PHP for the current shell.

If `php -v` doesn’t match, see §7 (PATH).

---

## 5) Install and verify Composer
- **Composer** is included: try `composer -V`.
- Alternatively, `herd composer -V` to invoke it via Herd.

If `composer` isn’t found, see §7 for PATH or always prefix with `herd`.

---

## 6) Manage multiple PHP versions
- **Global default**: choose it in Herd.
- **Per-command** via launcher:
  ```powershell
  herd php@8.3 -v
  herd php@8.2 -S 127.0.0.1:8000 -t public
  ```
- **Per project (IDE)**: point the IDE to the desired **PHP executable** (see VS Code/PhpStorm pages).

---

## 7) Add PHP to PATH (if needed)
If `php`/`composer` aren’t found:
1. **Control Panel → System → Advanced system settings → Environment Variables**.
2. In **System variables** → **Path** → **Edit**.
3. Add Herd’s bin (e.g., `C:\Users\<USER>\AppData\Roaming\Herd\bin`) **or** the specific version path shown by Herd.
4. Close & **reopen** terminal, then verify:
   - `where php`
   - `php -v`
   - `composer -V`

> Keep PATH **clean**: remove legacy entries (XAMPP/WAMP) to avoid conflicts.

---

## 8) Locate and edit `php.ini` (CLI)
- Each version in Herd has its own folder (e.g., `…\Herd\bin\php\8.x\`).
- If only `php.ini-development`/`php.ini-production` exist, copy one to `php.ini` and edit.

**Useful dev settings:**
```ini
display_errors = On
error_reporting = E_ALL
date.timezone = "Europe/London"  ; or your timezone

extension=curl
extension=mbstring
extension=intl
extension=fileinfo
extension=openssl
```

> Debugging with `php artisan serve` uses **CLI**, so Xdebug must be enabled in the **CLI** `php.ini` (see the Xdebug page).

---

## 9) Final checks
- `php -v` → correct version?
- `where php` → points to Herd?
- `composer -V` → available?
- `php -m` → required extensions loaded?

---

## 10) Common issues
**A) `php` not recognized** → PATH not updated; add correct path and reopen terminal.  
**B) Wrong version** → multiple PHPs in PATH; keep only Herd first. In IDE, set explicit executable.  
**C) Missing extensions** → enable in `php.ini` (remove `;`) and check with `php -m`.  
**D) Composer not found** → use `herd composer` or add to PATH.

---

## 11) Next steps
- **Page 02** – Enable **Xdebug** with the **Wizard** (DLL, `ext`, ini, pretty var_dump)  
- **Page 03** – **VS Code**: PHP Extension Pack, executable path, `launch.json`  
- **Page 04** – **PhpStorm**: Server, mapping, `PHP_IDE_CONFIG`  
- **Page 05** – Debug with **Laravel Sail** (Docker)  
- **Page 06** – Debug with **`php artisan serve`** (no Docker)
