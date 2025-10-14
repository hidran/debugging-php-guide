# PhpStorm – Full Setup: Server, Path Mappings, `PHP_IDE_CONFIG`, Xdebug 3

This page shows how to configure **PhpStorm** for reliable **Xdebug 3** debugging in two scenarios:
1) **No Docker** (e.g., `php artisan serve`)  
2) **Docker / Laravel Sail**

You’ll create a **Server**, set **path mappings** (when needed), use **`PHP_IDE_CONFIG`** to hint the server name, and fix common issues (port 9003, unknown server, competing IDEs).

> **Prerequisites**
> - Xdebug 3 active in the environment PHP runs (CLI for `artisan serve`, or container PHP).  
> - Know the host/port you browse (e.g., `127.0.0.1:8000`).

---

## 1) PhpStorm base settings
- Settings → PHP → Debug:
  - **Debug port**: `9003`
  - Enable **Start Listening for PHP Debug Connections** (telephone icon)
- Ensure no other IDE is listening on the same port.

---

## 2) Scenario A – No Docker (`php artisan serve`)
No path mappings needed (local files == executed files).

**Create Server**
- Settings → PHP → Servers → **+**
  - **Name**: `LocalLaravel`
  - **Host**: `127.0.0.1`
  - **Port**: `8000`
  - **Debugger**: Xdebug
  - **Use path mappings**: **OFF**

**Start & test**
- Terminal:
  ```bash
  php artisan serve --host=127.0.0.1 --port=8000
  ```
- PhpStorm: **Start Listening**
- Breakpoint in `routes/web.php` or a controller → open `http://127.0.0.1:8000/_debug`

**“Ignore connections from unknown servers”**
- If enabled, PhpStorm drops sessions not tied to a known Server.
- Either disable it or export a matching server name:
  ```bash
  # Before starting artisan serve:
  export PHP_IDE_CONFIG=serverName=LocalLaravel
  ```

---

## 3) Scenario B – Docker / Laravel Sail
With containers the paths differ (e.g., `/var/www/html`), so **path mappings are required**.

**Create Server**
- Settings → PHP → Servers → **+**
  - **Name**: `sail`
  - **Host**: `127.0.0.1` (match the URL you browse)
  - **Port**: `8000` (or the exposed one)
  - **Debugger**: Xdebug
  - ✅ **Use path mappings**: ON
    - Map **project root** → **`/var/www/html`** (Sail default)

**Hint the server name**
In the container env:
```yaml
environment:
  - PHP_IDE_CONFIG=serverName=sail
```
The value **must match** the PhpStorm Server **Name**.

**Xdebug in container**
- `xdebug.client_port=9003`
- `xdebug.client_host=host.docker.internal` (macOS/Windows) or `172.17.0.1` (Linux if needed)
- `xdebug.start_with_request = yes` or `trigger` (+ `?XDEBUG_TRIGGER=1`)

**Test**
- Start containers, listen on 9003, set a breakpoint, hit the URL.

If the log says “Connected…” but no break, it’s almost always **path mapping**.

---

## 4) Xdebug helper & trigger mode
With `xdebug.start_with_request = trigger`, enable the trigger:
- Query: `?XDEBUG_TRIGGER=1`
- Header: `XDEBUG_TRIGGER: 1`
- Cookie: **Xdebug helper** browser extension (set to **Xdebug v3**)

---

## 5) Quick checks & log
- PhpStorm listening, port 9003
- Xdebug active in the correct SAPI
- Host/URL matches the Server host
- Only one IDE listening

**Xdebug log** (temporary):
```ini
xdebug.log = /tmp/xdebug.log
xdebug.log_level = 7
```
- “Connected to debugging client” → IDE receives; if no break, mapping/breakpoint.
- “Could not connect…” → IDE not listening / wrong port / firewall.

---

## 6) Common issues
- **Unknown server** → disable the option or define a Server & use `PHP_IDE_CONFIG`.
- **No break** → ensure the line executes; put a break in an entry point.
- **Wrong path mappings (Docker)** → project root ↔ `/var/www/html` (adjust per image).
- **Port 9003 busy / competing IDE** → close the other IDE or change port consistently.
- **Wrong PHP/ini** → verify with `php --ri xdebug` (CLI) or `phpinfo()` (FPM).

---

## 7) Final checklist
- **No Docker**: Server `127.0.0.1:8000`, no mappings, optional `PHP_IDE_CONFIG`.
- **Sail**: Server, mappings ON, `PHP_IDE_CONFIG` matches, client host/port correct.
