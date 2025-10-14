# Debug con Laravel Sail (Docker): servizi, Xdebug nel container, mapping e test end‑to‑end

Questa pagina spiega come fare **debug** di un’app **Laravel** in **Docker** usando **Laravel Sail** e **Xdebug 3**. Vedremo come:
- aggiungere **servizi** (MariaDB, Valkey, Mailpit) al `docker-compose.yml`,
- **abilitare Xdebug** nel container PHP,
- configurare **PhpStorm** e **VS Code** (port 9003, path mappings),
- usare `PHP_IDE_CONFIG` per collegare le sessioni DBGp al **Server** giusto,
- fare **test end‑to‑end** con breakpoints in controller, service, job, test,
- risolvere i **problemi comuni** (porta, host, mapping, log).

> Prerequisiti: un progetto Laravel (11+ consigliato) con **Sail** installato (`php artisan sail:install`) e **pubblicato** (`php artisan sail:publish`). Docker + Docker Compose attivi.

---

## 1) Struttura base e avvio

Se non l’hai già fatto:
```bash
# Installa Sail nel progetto
php artisan sail:install

# (Consigliato) Pubblica i file per poterli personalizzare
php artisan sail:publish

# Avvia i container
./vendor/bin/sail up -d
```

Verifica che l’app risponda sull’host/porta esposta (spesso `http://127.0.0.1:80` o `:8000` se hai rimappato).

---

## 2) Aggiungere i servizi: MariaDB, Valkey, Mailpit

Apri `docker-compose.yml` e aggiungi o verifica le sezioni dei servizi. Esempio minimale:

```yaml
services:
  laravel.test:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:80"         # espone l'app su http://127.0.0.1:8000
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
      - "1025:1025"   # SMTP
      - "8025:8025"   # Web UI

volumes:
  mariadb_data:
```

Aggiorna `.env` in base ai servizi:
```
DB_CONNECTION=mysql
DB_HOST=mariadb
DB_PORT=3306
DB_DATABASE=app
DB_USERNAME=sail
DB_PASSWORD=password

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
REDIS_HOST=valkey
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="dev@example.com"
MAIL_FROM_NAME="Local Dev"
```

Riavvia i container dopo le modifiche:
```bash
./vendor/bin/sail down
./vendor/bin/sail up -d
```

---

## 3) Abilitare Xdebug nel container PHP

Con le immagini di Sail moderne, **Xdebug è già presente**. Assicurati che sia attivo in **mode** e **start** corretti.

### 3.1 Modalità Xdebug (env Sail)
Nel servizio `laravel.test` usa:
```
SAIL_XDEBUG_MODE=develop,debug
```

### 3.2 Avvio delle sessioni
Scegli uno dei due approcci:
- **Automatico**: avvio sempre
  ```ini
  xdebug.start_with_request = yes
  ```
- **Su trigger** (performance migliore quando non debuggiamo):
  ```ini
  xdebug.start_with_request = trigger
  ```
  e attiva via `?XDEBUG_TRIGGER=1` o estensione **Xdebug helper**.

> Se hai pubblicato i file (`sail:publish`), puoi trovare/creare la configurazione ini nella cartella `docker/` o in `/usr/local/etc/php/conf.d/*.ini` dentro al container.

### 3.3 Host/porta del client (IDE)
- **Porta (standard Xdebug 3)**: `xdebug.client_port = 9003`
- **Host** (dove gira l’IDE):
    - macOS/Windows: `xdebug.client_host = host.docker.internal`
    - Linux: se sopra non funziona, usa il **gateway Docker**: `172.17.0.1`

> Questi parametri puoi metterli in un ini personalizzato o in ENV, purché siano coerenti.

### 3.4 Verifica dentro al container
```bash
./vendor/bin/sail exec laravel.test php --ri xdebug | grep -E "Version|mode|start_with_request|client_host|client_port"
```

---

## 4) PhpStorm: Server, mapping, `PHP_IDE_CONFIG`

### 4.1 Crea il Server
**Settings → PHP → Servers → +**
- **Name**: `sail`
- **Host**: `127.0.0.1` (usa lo stesso host che usi nel browser)
- **Port**: `8000` (o la porta che hai esposto)
- **Debugger**: Xdebug
- ✅ **Use path mappings**: **ON**
    - Mappa **root progetto locale** → **`/var/www/html`** (Sail default)

### 4.2 “Indizio” per PhpStorm: `PHP_IDE_CONFIG`
Nel compose abbiamo messo:
```
PHP_IDE_CONFIG=serverName=sail
```
Il **serverName** **deve combaciare** con il **Name** del Server PhpStorm (`sail`). In questo modo, quando arriva una sessione DBGp, PhpStorm sa **quali mapping** applicare.

### 4.3 Ascolto e test
- PhpStorm: **Start Listening** (telefono) + Debug port **9003**
- Breakpoint su `routes/web.php` o un controller
- Visita `http://127.0.0.1:8000/_debug`

Se vedi nel log Xdebug “**Connected to debugging client**” ma PhpStorm non si ferma, quasi sempre è un **mapping errato**.

---

## 5) VS Code: Listen e pathMappings (se usi immagini custom)

Con Sail standard, spesso i percorsi sono coerenti grazie al mount `.:/var/www/html`. Se usi immagini/percorsi diversi, aggiungi i **pathMappings** in VS Code:

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

Poi: avvia **“Listen for Xdebug”**, metti un breakpoint, apri `http://127.0.0.1:8000/...`.

---

## 6) Debug di test, job e code (queue) dentro Sail

### 6.1 Test (Pest/PHPUnit)
```bash
./vendor/bin/sail test
# oppure, per forzare il collegamento Xdebug con start a trigger:
XDEBUG_MODE=debug ./vendor/bin/sail php -dxdebug.start_with_request=yes artisan test
```

### 6.2 Job/Queue con Valkey (Redis‑compat)
Se avvii un worker:
```bash
./vendor/bin/sail artisan queue:work
```
con `xdebug.start_with_request = yes`, Xdebug tenterà di collegarsi al tuo IDE quando il job esegue il codice col breakpoint. In modalità **trigger**, puoi impostare variabili d’ambiente solo per quel comando:
```bash
./vendor/bin/sail php -dxdebug.start_with_request=yes artisan queue:work
```

### 6.3 Comandi artisan personalizzati
Analogamente ai test:
```bash
XDEBUG_MODE=debug ./vendor/bin/sail php -dxdebug.start_with_request=yes artisan my:command
```

---

## 7) Diagnostica con il log Xdebug

Tieni aperto un tail del log **dentro al container**:
```bash
./vendor/bin/sail exec laravel.test sh -lc 'touch /tmp/xdebug.log && tail -f /tmp/xdebug.log'
```
- “**Connected to debugging client**” → la IDE riceve; se non si ferma → **mapping o breakpoint non raggiunto**.
- “**Could not connect to debugging client**” → la IDE **non** sta ascoltando / porta errata / `client_host` errato.

Disattiva/abbassa `xdebug.log_level` quando hai finito (scrive molto).

---

## 8) Problemi comuni e soluzioni

1) **Porta 9003 occupata / IDE concorrente**  
   Chiudi altre IDE o usa una porta custom **coerente** sia in IDE sia in `xdebug.client_port`.

2) **host.docker.internal non risolve (Linux)**  
   Usa `172.17.0.1` (gateway Docker) o configura una route; verifica con `ping` dal container.

3) **Breakpoint non scatta ma log “Connected…”**  
   Quasi sempre è **path mapping** in PhpStorm/VS Code. Verifica: project root ↔ `/var/www/html`.

4) **Unknown server in PhpStorm**
    - Disattiva l’opzione **Ignore connections from unknown servers**, **oppure**
    - Imposta `PHP_IDE_CONFIG=serverName=<Name>` uguale al Server definito.

5) **Configurazioni duplicate/ini errati**  
   Evita duplicare `zend_extension` o ini in conflitto. Controlla con `php --ri xdebug` dentro al container.

6) **Servizi non raggiungibili** (DB/Valkey/Mailpit)  
   Controlla host interni (`mariadb`, `valkey`, `mailpit`), porte, credenziali `.env`, stato container.

---

## 9) Checklist finale (Sail)

- `docker-compose.yml` con **MariaDB, Valkey, Mailpit** (se servono).
- `SAIL_XDEBUG_MODE=develop,debug`, `xdebug.start_with_request = yes|trigger`.
- `xdebug.client_port=9003`, `xdebug.client_host=host.docker.internal` (o `172.17.0.1` su Linux).
- PhpStorm: **Server** `sail`, **path mappings ON** (`/var/www/html`), **PHP_IDE_CONFIG=serverName=sail**.
- VS Code: **Listen** su 9003, `pathMappings` se usi percorsi custom.
- Breakpoint su entrypoint, **IDE in ascolto**, log Xdebug pronto per diagnosi.

Se tutto è coerente, il debug Docker con Sail è **stabile**: controller, service, job, test e perfino worker di coda si fermeranno dove chiedi.

---

**Prossima pagina:** Debug con `php artisan serve` (senza Docker): differenze chiave, setup rapido e troubleshooting mirato.
