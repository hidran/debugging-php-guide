# Debug with `php artisan serve` (no Docker): quick setup & troubleshooting

This page explains how to **debug** a Laravel project **without Docker**, using the built-in server (`php artisan serve`). It’s **simple**: no path mappings. But you must ensure Xdebug is enabled in **CLI**, not FPM/Apache.

> Prereqs:
> - Local PHP (Herd recommended on Windows)
> - Xdebug 3 active for **CLI** (`php --ri xdebug` shows Version/mode)
> - IDE: **PhpStorm** or **VS Code** listening on **port 9003**

---

## 1) Enable Xdebug in CLI
Find where CLI loads its ini and enable Xdebug there:
```powershell
php --ini
```
Edit the CLI ini (e.g. `...\Herd\bin\php\8.x\php.ini`):
```ini
zend_extension = php_xdebug.dll
xdebug.mode = develop,debug
xdebug.start_with_request = yes   ; or: trigger
xdebug.client_port = 9003
xdebug.client_host = 127.0.0.1
; xdebug.log = C:\Temp\xdebug.log
; xdebug.log_level = 7
```
Or enable on the fly:
```powershell
$env:XDEBUG_MODE="develop,debug"
php -dxdebug.start_with_request=yes artisan serve --host=127.0.0.1 --port=8000
```

---

## 2) Start the server
```powershell
php artisan serve --host=127.0.0.1 --port=8000
```

---

## 3) PhpStorm (no Docker)
- Settings → PHP → Debug: port **9003**, **Start Listening**.
- Settings → PHP → Servers → **+**
  - **Name**: `LocalLaravel`
  - **Host**: `127.0.0.1`
  - **Port**: `8000`
  - **Debugger**: Xdebug
  - **Use path mappings**: **OFF**
- If **Ignore unknown servers** is ON:
  ```powershell
  $env:PHP_IDE_CONFIG="serverName=LocalLaravel"
  ```
Set a breakpoint and open `http://127.0.0.1:8000/_debug`.

---

## 4) VS Code (no Docker)
- Install **PHP Debug**.
- `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    { "name": "Listen for Xdebug (Web)", "type": "php", "request": "launch", "port": 9003 },
    { "name": "Launch currently open script (CLI)", "type": "php", "request": "launch",
      "program": "${file}", "cwd": "${fileDirname}", "port": 9003,
      "runtimeArgs": ["-dxdebug.start_with_request=yes"],
      "env": {"XDEBUG_MODE": "debug,develop"} }
  ]
}
```
Start `artisan serve`, run **Listen for Xdebug (Web)**, open the URL.  
If using `trigger`, add `?XDEBUG_TRIGGER=1` or the browser extension.

---

## 5) Tests / CLI commands
```powershell
$env:XDEBUG_MODE="debug"
php -dxdebug.start_with_request=yes artisan test
php -dxdebug.start_with_request=yes artisan my:command
```

---

## 6) Troubleshooting
- No break → IDE not listening, wrong start mode, line not executed, host mismatch.
- “Could not connect…” → IDE not listening / wrong/busy port / other IDE stealing connection.
- Xdebug not in CLI → fix ini; `php --ri xdebug` must show Version & mode.
- PhpStorm grey break → define Server or set `PHP_IDE_CONFIG`.
- Performance → use `trigger`.

---

## 7) Recap
- Enable Xdebug in **CLI**.  
- `artisan serve` on **127.0.0.1:8000**.  
- IDE listens on **9003**.  
- No path mapping needed.  
- Use **trigger** if you want opt‑in debugging.
