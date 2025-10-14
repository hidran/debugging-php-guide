# VS Code – Full Setup: PHP Extension Pack, PHP executable path, launch.json

This page explains how to configure **Visual Studio Code** for **PHP development and debugging** on **Windows**, after installing PHP with **Herd** and enabling **Xdebug**. You’ll get **IntelliSense**, linting, and a **working debugger** for both **CLI scripts** and **web requests** (`php artisan serve`).

> **Prerequisites**
> - `php -v`, `where php` → PHP reachable (Herd)
> - `php --ri xdebug` → Xdebug 3 active in **CLI** (Version, mode, port 9003)
> - A PHP/Laravel project opened in VS Code

---

## 1) Install extensions
- **PHP Intelephense**
- **PHP Debug** (`xdebug.php-debug`)
- (Optional) **EditorConfig**, **Error Lens**, **DotENV**

Restart VS Code if prompted.

---

## 2) Set the **PHP executable path**
**Settings UI**
1. `Ctrl+,` → search **php validate executable path**
2. Set, e.g.:
   ```
   C:\Users\<USER>\AppData\Roaming\Herd\bin\php\php.exe
   ```

**Or via file** `.vscode/settings.json`:
```json
{
  "php.validate.enable": true,
  "php.validate.executablePath": "C:\\Users\\<USER>\\AppData\\Roaming\\Herd\\bin\\php\\php.exe",
  "intelephense.environment.phpVersion": "8.3",
  "editor.formatOnSave": true
}
```

**Verify in terminal (View → Terminal):**
```powershell
php -v
```
It must match Herd’s PHP.

---

## 3) Create `.vscode/launch.json`
Two baseline configurations:

- **Listen for Xdebug (Web)** → listen for DBGp (for `artisan serve`)
- **Launch currently open script (CLI)** → run & debug the open file

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug (Web)",
      "type": "php",
      "request": "launch",
      "port": 9003
    },
    {
      "name": "Launch currently open script (CLI)",
      "type": "php",
      "request": "launch",
      "program": "${file}",
      "cwd": "${fileDirname}",
      "port": 9003,
      "runtimeArgs": [
        "-dxdebug.start_with_request=yes"
      ],
      "env": {
        "XDEBUG_MODE": "debug,develop"
      }
    }
  ]
}
```

---

## 4) Web debug with `php artisan serve`
1. In terminal:
   ```powershell
   php artisan serve --host=127.0.0.1 --port=8000
   ```
2. VS Code → **Run & Debug** → **Listen for Xdebug (Web)** → Start.
3. Put a breakpoint in `routes/web.php`:
   ```php
   Route::get('/_debug', function () {
       $x = ['hello' => 'world'];
       return 'ok';
   });
   ```
4. Open `http://127.0.0.1:8000/_debug`.

> If `xdebug.start_with_request = trigger`, use `?XDEBUG_TRIGGER=1` or the **Xdebug helper** browser extension. With `yes`, no trigger needed.

---

## 5) CLI debug (single file, commands)
Open the file, set a breakpoint, run **Launch currently open script (CLI)**.  
It starts PHP with `XDEBUG_MODE=debug,develop` and `-dxdebug.start_with_request=yes`.

Examples: small scripts, artisan commands (point `program` to `artisan` and pass args via `args`).

---

## 6) Useful variants
**Tests (Pest/PHPUnit):**
```powershell
$env:XDEBUG_MODE="debug"
php -dxdebug.start_with_request=yes artisan test
```

**Built-in PHP server (non-Laravel apps):**
```json
{
  "name": "Built-in PHP server (debug)",
  "type": "php",
  "request": "launch",
  "runtimeArgs": [
    "-dxdebug.mode=debug",
    "-dxdebug.start_with_request=yes",
    "-S", "127.0.0.1:8001",
    "-t", "."
  ],
  "cwd": "${workspaceFolder}/public",
  "port": 9003
}
```

**Profiling (performance):**
```json
{
  "name": "Profile current script",
  "type": "php",
  "request": "launch",
  "program": "${file}",
  "cwd": "${fileDirname}",
  "port": 9003,
  "env": { "XDEBUG_MODE": "profile" }
}
```

---

## 7) Breakpoint tips
- Place initial breakpoints at **certain entry points** (route, controller).
- Use **Conditional Breakpoints** and **Logpoints** to narrow down flows.
- Learn shortcuts: step into/over/out (F11/F10/Shift+F11).

---

## 8) Common pitfalls
- **Competing IDE** on 9003 (PhpStorm): close one or change ports consistently.
- **Wrong PHP** in VS Code: fix `"php.validate.executablePath"` and reopen the terminal.
- **Xdebug not loaded in CLI**: `php --ri xdebug` must show Version & mode.
- **Host/URL mismatch**: prefer **127.0.0.1** consistently.
- **Trigger missing**: with trigger mode, add `?XDEBUG_TRIGGER=1` or cookie/header.
- **Grey breakpoints**: file/line not executed.

---

## 9) Troubleshooting
- Listener active? port 9003?
- `start_with_request = yes` (or trigger used)?
- Is the line actually executed?  
- **Xdebug log** (temporarily in CLI `php.ini`):
  ```ini
  xdebug.log = C:\Temp\xdebug.log
  xdebug.log_level = 7
  ```
  - “Connected to debugging client” but no break → unreachable breakpoint or IDE config.
  - “Could not connect to debugging client” → IDE not listening / wrong port / port busy.

---

## 10) Recap
- Install **PHP Debug** + **Intelephense**.
- Point executable to Herd’s PHP.
- Create `launch.json` with **Web** and **CLI** configs.
- For Laravel: `artisan serve` on **127.0.0.1:8000**, listen on 9003, break on route/controller.
