# Debug with Laravel Sail (Docker): services, Xdebug in container, mappings, end-to-end tests

This page explains how to **debug** a **Laravel** app in **Docker** using **Laravel Sail** and **Xdebug 3**. You’ll:
- add **services** (MariaDB, Valkey, Mailpit) to `docker-compose.yml`,
- **enable Xdebug** in the PHP container,
- configure **PhpStorm/VS Code** (port 9003, path mappings),
- use **`PHP_IDE_CONFIG`** to bind DBGp sessions to the right Server,
- run **end-to-end tests** with breakpoints,
- fix **common issues** (port, host, mappings, logs).

> Prereqs: Laravel project (11+ recommended) with **Sail** installed (`php artisan sail:install`) and **published** (`php artisan sail:publish`). Docker + Compose available.

---

## 1) Base structure & start
```bash
php artisan sail:install
php artisan sail:publish
./vendor/bin/sail up -d
```
Browse the exposed host/port (e.g., `http://127.0.0.1:8000`).

---

## 2) Add services: MariaDB, Valkey, Mailpit
In `docker-compose.yml`:

```yaml
services:
  laravel.test:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:80"
    environment:
      - APP_ENV=local
      - PHP_IDE_CONFIG=serverName=sail
      - SAIL_XDEBUG_MODE=develop,debug
    volumes:
      - ./:/var/www/html
    depends_on:
      - mariadb
      - valkey
      - mailpit

  mariadb:
    image: mariadb:11
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: app
      MYSQL_USER: sail
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql

  valkey:
    image: valkey/valkey:latest
    ports:
      - "6379:6379"

  mailpit:
    image: axllent/mailpit:latest
    ports:
      - "1025:1025"
      - "8025:8025"

volumes:
  mariadb_data:
```

Update `.env` accordingly (DB host `mariadb`, redis `valkey`, mail `mailpit`). Restart:
```bash
./vendor/bin/sail down
./vendor/bin/sail up -d
```

---

## 3) Enable Xdebug in the container
**Modes (env):**  
`SAIL_XDEBUG_MODE=develop,debug`

**Start:**  
- `xdebug.start_with_request = yes`, or  
- `trigger` + `?XDEBUG_TRIGGER=1` / Xdebug helper.

**Client:**  
- `xdebug.client_port = 9003`  
- `xdebug.client_host = host.docker.internal` (macOS/Windows) or `172.17.0.1` (Linux).

**Verify inside container:**
```bash
./vendor/bin/sail exec laravel.test php --ri xdebug | grep -E "Version|mode|start_with_request|client_host|client_port"
```

---

## 4) PhpStorm: Server, mappings, `PHP_IDE_CONFIG`
- Server **Name**: `sail`
- **Host**: `127.0.0.1`, **Port**: `8000`
- ✅ **Use path mappings**: ON → map project root → **`/var/www/html`**
- In compose: `PHP_IDE_CONFIG=serverName=sail` (name must match).

Listen, set a breakpoint, hit the URL. If the log says “Connected…” but no break → fix **mappings**.

---

## 5) VS Code: Listen + pathMappings (custom images)
If your container path differs, set `pathMappings`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug (Sail)",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        "/var/www/html": "${workspaceFolder}"
      }
    }
  ]
}
```

---

## 6) Debug tests, queues, commands
**Tests:**
```bash
./vendor/bin/sail test
# or force connect:
XDEBUG_MODE=debug ./vendor/bin/sail php -dxdebug.start_with_request=yes artisan test
```

**Queue worker:**
```bash
./vendor/bin/sail artisan queue:work
# or with forced start:
./vendor/bin/sail php -dxdebug.start_with_request=yes artisan queue:work
```

**Custom commands:**
```bash
XDEBUG_MODE=debug ./vendor/bin/sail php -dxdebug.start_with_request=yes artisan my:command
```

---

## 7) Xdebug logs (container)
```bash
./vendor/bin/sail exec laravel.test sh -lc 'touch /tmp/xdebug.log && tail -f /tmp/xdebug.log'
```
- “Connected…” → IDE receives; if no break → mappings or unreachable breakpoint.
- “Could not connect…” → IDE not listening / wrong port / wrong client host.

---

## 8) Common issues
- **Port 9003 busy / competing IDE** → close others or change port consistently.
- **host.docker.internal on Linux** → use `172.17.0.1` (Docker gateway).
- **No break but connected** → fix **path mappings**.
- **Unknown server (PhpStorm)** → disable the option or set `PHP_IDE_CONFIG` to match the Server name.
- **Duplicate ini / wrong Xdebug** → ensure only one `zend_extension` and correct ini in use.

---

## 9) Final checklist
- Services up (MariaDB, Valkey, Mailpit) as needed.  
- `SAIL_XDEBUG_MODE=develop,debug`, `start_with_request=yes|trigger`.  
- `client_port=9003`, `client_host` correct.  
- PhpStorm: Server **sail**, mappings ON, `PHP_IDE_CONFIG` matches.  
- VS Code: listen on 9003, `pathMappings` if paths differ.
