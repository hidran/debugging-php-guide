# Xdebug via Wizard on Windows (Herd) – Pro Installation & Config

This guide shows how to install and configure **Xdebug 3 on Windows** when PHP is installed with **Herd**. You’ll use the official **Xdebug Wizard** to download the **correct DLL**, copy it to PHP’s **ext** directory, and configure the **CLI `php.ini`** (used by `php artisan serve`). We’ll also cover **prettier var_dump**, **`start_with_request`**, **trigger mode**, and **troubleshooting**.

---

## 1) Verify CLI version & INI
Make sure you’re on **CLI**, not FPM/Apache:

```powershell
php -v
php --ini
where php
```

- `php --ini` shows where the **CLI** ini files are (e.g., `C:\Users\<USER>\AppData\Roaming\Herd\bin\php\8.x\php.ini`).
- **Note** the path: that’s where we’ll enable Xdebug.

---

## 2) Use the official Wizard for the correct DLL
1. Run:
   ```powershell
   php -i > phpinfo.txt
   ```
2. Open **https://xdebug.org/wizard** and paste the **entire** `phpinfo.txt` content.
3. The Wizard tells you which **DLL** to download (e.g., `php_xdebug-3.x.x-8.3-vs16-nts-x64.dll`) and where to place it.
4. Optionally rename it to `php_xdebug.dll` for a simpler `zend_extension` line.

---

## 3) Copy the DLL to `ext`
Typical Herd path:
```
C:\Users\<USER>\AppData\Roaming\Herd\bin\php\8.x\ext\
```
Place `php_xdebug.dll` there (or the exact filename from the Wizard).

---

## 4) Enable Xdebug in the **CLI** `php.ini`
Open the CLI ini you noted in §1 and add these lines **once**:

```ini
; === Xdebug – module enable ===
zend_extension = php_xdebug.dll

; === Recommended dev modes ===
xdebug.mode = develop,debug
xdebug.start_with_request = yes

; === IDE client ===
xdebug.client_port = 9003
xdebug.client_host = 127.0.0.1

; === (Optional) Diagnostic log ===
; xdebug.log = C:\Temp\xdebug.log
; xdebug.log_level = 7
```

**Notes**
- If you didn’t rename the DLL, use the **exact** filename or full path the Wizard suggested.
- Xdebug 3 default port is **9003** (not 9000).

Close & **reopen** your terminal.

---

## 5) Confirm Xdebug is loaded
```powershell
php --ri xdebug
```
You should see **Version**, `xdebug.mode => develop,debug`, `xdebug.start_with_request => yes`, `xdebug.client_port => 9003`.

If you get “Xdebug not found”, re-check **DLL path/name**, `zend_extension`, and that you edited the **CLI** ini.

---

## 6) Start modes
**Always on (easy for setup)**:
```ini
xdebug.start_with_request = yes
```
**Trigger only (better perf):**
```ini
xdebug.start_with_request = trigger
```
Activate via:
- Query: `?XDEBUG_TRIGGER=1`
- Header: `XDEBUG_TRIGGER: 1`
- Cookie: browser extension **Xdebug helper** (set to **Xdebug v3**).

---

## 7) Prettier `var_dump`
```ini
xdebug.var_display_max_children = 256
xdebug.var_display_max_data     = 1024
xdebug.var_display_max_depth    = 5
```

---

## 8) Connect the IDE (VS Code / PhpStorm)
**VS Code**
- Install **PHP Debug** (xdebug.php-debug).
- Create `.vscode/launch.json` with **Listen for Xdebug** on **port 9003**.
- For web debug with `php artisan serve`:
  ```powershell
  php artisan serve --host=127.0.0.1 --port=8000
  ```
  Then open `http://127.0.0.1:8000/...` with VS Code listening.

**PhpStorm**
- Settings → PHP → Debug: **9003**, **Start Listening**.
- Without Docker: no path mappings. Define a **Server** `127.0.0.1:8000` if “Ignore unknown servers” is enabled.

---

## 9) Quick debug test
1. In a Laravel project add:
   ```php
   // routes/web.php
   Route::get('/_debug', function () {
       $a = ['hello' => 'world'];
       return 'ok';
   });
   ```
2. Put a breakpoint on `return`.
3. Start:
   ```powershell
   php artisan serve --host=127.0.0.1 --port=8000
   ```
4. Ensure the IDE listens on 9003 and open `http://127.0.0.1:8000/_debug`.

With `start_with_request = yes` no trigger is needed; with `trigger`, add `?XDEBUG_TRIGGER=1`.

---

## 10) Troubleshooting
- **“Xdebug not found”** → wrong DLL (TS/NTS, x86/x64) or wrong ini (FPM/Apache vs CLI).  
- **“Could not connect to debugging client”** → IDE not listening / wrong port / firewall / host.  
- **No break but connected** → wrong path mapping (Docker) or breakpoint not reached.  
- **Ugly var_dump** → ensure `mode` contains `develop` and adjust display limits.

---

## 11) TL;DR
- Use Wizard for the correct DLL; enable in **CLI** `php.ini` via `zend_extension`.  
- `xdebug.mode=develop,debug`, choose `start_with_request=yes|trigger`.  
- IDE on **9003**, one IDE at a time.  
- With `artisan serve` no path mappings are needed.
