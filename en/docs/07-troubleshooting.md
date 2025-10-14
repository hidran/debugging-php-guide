# Advanced Troubleshooting – Xdebug, VS Code, PhpStorm, Sail & artisan serve

This page collects **deep diagnostics** and **fixes** for common debugging issues with **Xdebug 3** locally (Windows + Herd) and in containers (Laravel Sail/Docker).

---

## 1) Quick diagnosis flow
1. **IDE listening?**  
   - VS Code: **Listen for Xdebug** started on 9003  
   - PhpStorm: **Start Listening** (telephone icon)  
2. **Xdebug active in the right SAPI?**  
   - CLI: `php --ri xdebug`  
   - Sail: `sail exec laravel.test php --ri xdebug`  
3. **Does Xdebug connect?**  
   - Enable log and check “Connected…” vs “Could not connect…”  
4. **Breakpoint reachable?**  
   - Put it in `routes/web.php` or `public/index.php`  
5. **Correct Host/Port/Server?**  
   - Use `127.0.0.1`. PhpStorm Server name matches `PHP_IDE_CONFIG`. Docker mappings correct.

---

## 2) Xdebug log
**Windows (CLI):**
```ini
xdebug.log = C:\Temp\xdebug.log
xdebug.log_level = 7
```
**Docker/Sail:**
```bash
./vendor/bin/sail exec laravel.test sh -lc 'touch /tmp/xdebug.log && tail -f /tmp/xdebug.log'
```

- “Connected to debugging client” → IDE receives; if no break → mapping or breakpoint.
- “Could not connect…” → IDE not listening / wrong port / firewall.

Disable the log after debugging.

---

## 3) Port 9003 & conflicts
**Windows:**
```powershell
netstat -ano | findstr :9003
```
**macOS/Linux:**
```bash
lsof -i :9003 || ss -ltnp | grep 9003
```
Close the conflicting app or change the port **consistently** in IDE and `xdebug.client_port`.

Test reachability from container (Sail):
```bash
./vendor/bin/sail exec laravel.test sh -lc 'nc -zv host.docker.internal 9003 || nc -zv 172.17.0.1 9003 || echo "port closed"'
```

---

## 4) Hosts
- Local (no Docker): prefer **127.0.0.1** consistently.
- Docker/Sail: `host.docker.internal` (macOS/Windows) or `172.17.0.1` (Linux).

Ensure browser URL host matches the PhpStorm Server host.

---

## 5) PhpStorm – Unknown servers
If **Ignore connections from unknown servers** is ON, PhpStorm drops sessions not matching a Server. Either disable it or define a Server and pass:
```bash
# local
$env:PHP_IDE_CONFIG="serverName=LocalLaravel"
# docker
PHP_IDE_CONFIG=serverName=sail
```
The value must match the Server **Name**.

---

## 6) Path mappings
- No Docker: **no mappings**.  
- Sail: **required** → map project root ↔ **/var/www/html** (or your container path).

Symptom: “Connected…” in log, but no break in IDE → mapping issue.

---

## 7) Start modes
- `start_with_request = yes` → simplest.
- `trigger` → requires `?XDEBUG_TRIGGER=1` or cookie/header (Xdebug helper).

---

## 8) Verify the right PHP
Multiple installs cause confusion. Check:
```powershell
where php
php -v
php --ini
```
In VS Code fix `"php.validate.executablePath"`. In PhpStorm check the interpreter/CLI path. In Docker, always verify inside the container.

---

## 9) Test routes & scripts
**Laravel route:**
```php
Route::get('/_debug', function () {
    $x = ['hello' => 'world', 't' => time()];
    return 'ok';
});
```
**Minimal CLI script:**
```php
<?php
foreach (range(0, 3) as $i) {
    $x = $i * 2; // breakpoint
    echo "i=$i, x=$x\n";
}
```

---

## 10) Firewall / AV / VPN
Allow inbound connections on 9003 for your IDE. Some VPNs block loopback/ports—try disabling them temporarily.

---

## 11) Typical problems & quick fixes
- **Xdebug not found** → wrong DLL or wrong ini (FPM vs CLI). Use the **Wizard** and edit the **CLI** ini.
- **Could not connect** → IDE not listening / port wrong/busy / firewall / wrong host.
- **Connected but no break** → mappings or unreachable line.
- **PhpStorm ignores session** → define Server or disable “unknown servers” option.
- **VS Code runs wrong PHP** → fix executable path and reopen terminal.

---

## 12) Final checklist
- One IDE listening on **9003**.  
- `xdebug.mode=develop,debug`, `start_with_request=yes|trigger`.  
- Correct host (`127.0.0.1`, `host.docker.internal`/`172.17.0.1`).  
- PhpStorm: Server defined + `PHP_IDE_CONFIG`.  
- Docker: correct **path mappings**.  
- Test with `/_debug` route or minimal script.  
- Disable the Xdebug log after using it.
