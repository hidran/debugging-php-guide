# PhpStorm – Setup completo: Server, Path Mapping, `PHP_IDE_CONFIG` e Debug con Xdebug 3

Questa pagina spiega come configurare **PhpStorm** per un debugging affidabile con **Xdebug 3** in due scenari:
1) **Senza Docker** (es. `php artisan serve`),
2) **Con Docker / Laravel Sail** (container).

Imparerai a creare il **Server** in PhpStorm, impostare i **path mappings** (quando servono), usare la variabile d’ambiente **`PHP_IDE_CONFIG`** per identificare il server giusto e risolvere i problemi più comuni (porta 9003, “unknown server”, doppie IDE in ascolto).

> **Prerequisiti**
> - Xdebug 3 attivo per **CLI** (se usi `artisan serve`) o per l’ambiente in cui gira PHP. Verifica con `php --ri xdebug` o `phpinfo()`.
> - Conosci l’host/porta con cui raggiungi l’app (es. `127.0.0.1:8000` o `http://localhost:8080`).

---

## 1) Impostazioni base di PhpStorm per Xdebug 3

1. Apri **Settings → PHP → Debug**
    - **Debug port**: `9003` (standard Xdebug 3)
    - Lascia attivo **Can accept external connections** (icona “telefono” in alto: **Start Listening**)
2. **Disattiva** altri debugger concorrenti su 9003 (es. VS Code aperto). Una sola IDE deve ascoltare.

> Suggerimento: assegna scorciatoia rapida al toggle “Start Listening for PHP Debug Connections”.

---

## 2) Scenario A – **Senza Docker** (es. `php artisan serve`)

Con `php artisan serve` usi la **CLI** che espone un mini server HTTP. In questo caso **non servono path mappings**, perché i file sul disco sono gli stessi eseguiti da PHP.

### 2.1 Crea il Server in PhpStorm
- **Settings → PHP → Servers → +**
    - **Name**: `LocalLaravel` (o un nome a scelta)
    - **Host**: `127.0.0.1`
    - **Port**: `8000` (o quello che usi con `artisan serve`)
    - **Debugger**: Xdebug
    - **Use path mappings**: **OFF** (non necessario in locale)

### 2.2 Avvia il server e ascolta
- Terminale:
  ```bash
  php artisan serve --host=127.0.0.1 --port=8000
  ```
- In PhpStorm: **Start Listening** (telefono acceso).
- Aggiungi un **breakpoint** in `routes/web.php` o in un controller d’ingresso e apri `http://127.0.0.1:8000/_debug` (o la tua rotta).

### 2.3 Opzione “Ignore connections from unknown servers”
Se **attiva**, PhpStorm **ignora** le sessioni DBGp non mappate a un Server noto. In locale puoi:
- **Disattivarla** (più permissivo), **oppure**
- Lasciarla attiva ma **definire il Server** come sopra e, per sicurezza, esportare il **server name**:

```bash
# Prima di avviare artisan serve, opzionale ma utile:
export PHP_IDE_CONFIG=serverName=LocalLaravel
```

> Il nome nella variabile deve **combaciare** con il **Name** del Server in PhpStorm.

---

## 3) Scenario B – **Con Docker / Laravel Sail**

Con container, i file nel container stanno in un percorso diverso (es. `/var/www/html`). Qui i **path mappings sono OBBLIGATORI**.

### 3.1 Crea il Server in PhpStorm
- **Settings → PHP → Servers → +**
    - **Name**: `sail` (o uno a tua scelta)
    - **Host**: `127.0.0.1` (o `localhost`, **usa lo stesso host dell’URL**)
    - **Port**: quella esposta dal compose (es. `8000` → `http://127.0.0.1:8000`)
    - **Debugger**: Xdebug
    - ✅ **Use path mappings**: **ON**
        - Mappa la **root del progetto locale** → **`/var/www/html`** (per Sail standard)
        - Se hai sottocartelle/volumi diversi, aggiungi le mappature necessarie

### 3.2 Passa il `serverName` con `PHP_IDE_CONFIG`
Nel servizio dell’app (container PHP) passa una variabile d’ambiente che identifica il Server PhpStorm:

```yaml
environment:
  - PHP_IDE_CONFIG=serverName=sail
```

> Il valore `sail` deve **coincidere** con **Name** del Server creato in PhpStorm. Così, quando Xdebug invia file/uri, PhpStorm sa quale mapping applicare.

### 3.3 Impostazioni Xdebug nel container
- **Porta**: `xdebug.client_port=9003`
- **Host** (client IDE):
    - macOS/Windows: `xdebug.client_host=host.docker.internal`
    - Linux: `xdebug.client_host=172.17.0.1` *(gateway Docker, se l’host sopra non funziona)*
- **Avvio**:
    - `xdebug.start_with_request = yes` (più semplice) **oppure**
    - `xdebug.start_with_request = trigger` + `?XDEBUG_TRIGGER=1` / Xdebug helper

> Con Sail: `SAIL_XDEBUG_MODE=develop,debug` per assicurare le modalità giuste.

### 3.4 Avvio e test
1. Avvia i container (Sail o docker compose).
2. **PhpStorm in ascolto** su 9003.
3. Breakpoint in un entrypoint (es. route).
4. Apri l’URL `http://127.0.0.1:8000/...` (o la porta esposta nel compose).

Se i breakpoint non si fermano ma il log dice “**Connected to debugging client**”, è quasi sempre un **problema di mapping** (percorso container ≠ percorso locale).

---

## 4) Xdebug helper (browser) & modalità trigger

Se in `php.ini` hai `xdebug.start_with_request = trigger`, devi **abilitare il trigger**:
- **Query string**: `?XDEBUG_TRIGGER=1`
- **Header**: `XDEBUG_TRIGGER: 1`
- **Cookie**: estensione browser **Xdebug helper** (imposta **Xdebug 3**).

> In alternativa, usa `start_with_request = yes` durante il setup per ridurre le variabili in gioco.

---

## 5) Verifiche rapide & log

- **PhpStorm**: telefono su **ON**, **Debug port = 9003**.
- **Xdebug**: attivo e coerente (CLI o FPM a seconda del caso).
- **Host & URL**: usa **sempre lo stesso** (preferibilmente `127.0.0.1`).
- **Una sola IDE** in ascolto su 9003.

**Log Xdebug** (attivalo temporaneamente):
```ini
xdebug.log = /tmp/xdebug.log   ; su Linux/macOS/Docker
xdebug.log_level = 7
; su Windows: C:\Temp\xdebug.log (crea la cartella se manca)
```
- “**Connected to debugging client**” → PhpStorm riceve: se non si ferma, è mapping o breakpoint non raggiunto.
- “**Could not connect to debugging client**” → PhpStorm non ascolta / porta errata / firewall.

---

## 6) Errori comuni (e soluzioni)

1) **“Unknown server” ignorato da PhpStorm**
    - Disattiva **Ignore connections from unknown servers** **oppure** crea il Server corretto e passa `PHP_IDE_CONFIG=serverName=<Name>`.

2) **Breakpoint non raggiunto**
    - Mettilo su un punto di ingresso sicuro (route, controller, middleware).
    - Verifica che la richiesta **passi davvero** da quel file/linea.

3) **Path mappings errati (Docker)**
    - Mappa project root → `/var/www/html` (Sail default).
    - Se usi Nginx/Apache personalizzati o volumi particolari, assicurati che il **percorso container** corrisponda a quello che Xdebug invia.

4) **Porta 9003 occupata o IDE concorrente**
    - Chiudi l’altra IDE o cambia porta **ovunque** (PhpStorm e `xdebug.client_port`).

5) **PHP/ini sbagliato**
    - In locale: `php --ri xdebug` deve mostrare versione/mode della **CLI**; con FPM/Apache verifica via `phpinfo()`.

6) **host.docker.internal** su Linux non risolve
    - Usa `172.17.0.1` (gateway Docker) o crea una route dedicata; verifica con `ping` dal container.

---

## 7) Checklist finale

- **Senza Docker**
    - Server: `127.0.0.1:8000`, **no** path mappings.
    - (Se necessario) `PHP_IDE_CONFIG=serverName=LocalLaravel`.
    - Xdebug CLI attivo, `start_with_request=yes` (o trigger).

- **Con Docker/Sail**
    - Server: `127.0.0.1:<porta esposta>`.
    - **Path mappings ON**: root → `/var/www/html`.
    - `PHP_IDE_CONFIG=serverName=<Name>` **corrisponde** a PhpStorm.
    - Xdebug client host/port corretti (9003; `host.docker.internal`/gateway).

Se tutto è a posto, i breakpoint devono fermarsi in **controller**, **service**, **job**, **test**.

---

## 8) Riferimenti rapidi

- Xdebug 3 – porte e trigger: 9003, `start_with_request = yes|trigger`, `XDEBUG_TRIGGER=1`
- PhpStorm – **Servers**: host+port devono **combaciare** con l’URL
- Docker/Sail – **path mappings** e `PHP_IDE_CONFIG` sono cruciali

---

**Prossima pagina:** Debug con **Laravel Sail** (Docker): servizi, Xdebug nel container, mapping e test end‑to‑end.
