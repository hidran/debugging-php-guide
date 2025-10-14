# Troubleshooting avanzato – Xdebug, VS Code, PhpStorm, Sail e artisan serve

Questa pagina raccoglie **diagnostica profonda** e **soluzioni** ai problemi più comuni del debugging con **Xdebug 3** in locale (Windows con Herd) e in container (Laravel Sail/Docker). Segui i **check veloci**, poi entra nei casi specifici.

---

## 1) Flow di diagnosi rapido (decision tree)

1. **IDE in ascolto?**
    - VS Code: verifica che la config **Listen for Xdebug** sia **avviata** (port 9003).
    - PhpStorm: “**Start Listening for PHP Debug Connections**” attivo (icona telefono).  
      → Se **no**, accendi il listener e riprova.

2. **La richiesta arriva a Xdebug?**
    - CLI: `php --ri xdebug` → deve mostrare **Version**, `mode`, `start_with_request`.
    - Web (artisan serve): Xdebug è abilitato nella **CLI**.
    - Docker/Sail: `sail exec laravel.test php --ri xdebug`.  
      → Se **no**, abilita/installa Xdebug nel **giusto SAPI**.

3. **Xdebug si collega all’IDE?**
    - Attiva il **log** e cerca “**Connected to debugging client**” o “**Could not connect**”.  
      → Se **connected** ma **niente break**: mapping/breakpoint non raggiunto.  
      → Se **could not connect**: IDE non ascolta, porta sbagliata, firewall o host errato.

4. **Breakpoint in un punto eseguito?**
    - Metti un break in **routes/web.php** o `public/index.php` (entrypoint).
    - Verifica che l’URL/command esegua **davvero** quella riga.

5. **Host/Porta/Server corretti?**
    - Usa `http://127.0.0.1` (non `localhost`) e porta attesa.
    - PhpStorm: Server **Name** combacia con `PHP_IDE_CONFIG`?
    - Docker: path mapping **/var/www/html** ↔ root progetto?

---

## 2) Abilitare e leggere il log di Xdebug

### 2.1 Local Windows (CLI / artisan serve)
Nel `php.ini` **CLI**:
```ini
xdebug.log = C:\Temp\xdebug.log
xdebug.log_level = 7
```
Crea la cartella `C:\Temp\` se non esiste. Poi **riavvia** il terminale/server.

**Messaggi chiave**
- `Connected to debugging client` → l’IDE ha ricevuto la connessione.
- `Could not connect to debugging client` → l’IDE non ascolta / porta sbagliata / firewall / porta occupata.

> Ricorda di **disattivare** il log quando hai finito.

### 2.2 Docker / Sail
```bash
./vendor/bin/sail exec laravel.test sh -lc 'touch /tmp/xdebug.log && tail -f /tmp/xdebug.log'
```
In alternativa, scrivi il log in `/tmp/xdebug.log` via ini nel container.

---

## 3) Porte e conflitti (9003)

### 3.1 Verifica porta occupata
- **Windows (PowerShell)**:
  ```powershell
  netstat -ano | findstr :9003
  # oppure
  Get-NetTCPConnection -LocalPort 9003 -State Listen
  ```
- **macOS/Linux**:
  ```bash
  lsof -i :9003 || ss -ltnp | grep 9003
  ```

Se un’altra app/IDE usa 9003, **chiudila** o cambia porta **coerentemente** in IDE e in `xdebug.client_port`.

### 3.2 Test di reachability dal container
```bash
./vendor/bin/sail exec laravel.test sh -lc 'apk add --no-cache curl >/dev/null 2>&1 || true; nc -zv host.docker.internal 9003 || nc -zv 172.17.0.1 9003 || echo "port closed"'
```
Se non raggiunge l’host, rivedi `xdebug.client_host` o firewall.

---

## 4) Host corretti (127.0.0.1 vs localhost, Docker)

- **Locale (senza Docker)**: preferisci **127.0.0.1** ovunque (server e browser).
- **Docker/Sail**:
    - macOS/Windows: `xdebug.client_host = host.docker.internal`
    - Linux: usa il gateway es. `172.17.0.1` se l’host di cui sopra non risolve

> Assicurati che l’**URL** nel browser **combaci** con **Host/Port** configurati nel Server PhpStorm.

---

## 5) PhpStorm – “Ignore connections from unknown servers”

Se attivo, blocca le sessioni DBGp non mappate a un **Server** noto. Soluzioni:
- **Disattivalo** temporaneamente **oppure**
- Definisci un **Server** coerente e passa il nome via env:
  ```bash
  # Locale (PowerShell)
  $env:PHP_IDE_CONFIG="serverName=LocalLaravel"
  # Docker Compose/Sail (nel servizio app)
  PHP_IDE_CONFIG=serverName=sail
  ```
Il **Name** del Server in PhpStorm deve **combaciare** con `serverName`.

---

## 6) Path mappings (quando servono e quando no)

- **Senza Docker (artisan serve)**: **NON** servono path mappings. Disattivali.
- **Con Docker/Sail**: **OBBLIGATORI**. Mappa **root progetto** → **`/var/www/html`** (default Sail).  
  Se stai usando un **Dockerfile**/Nginx/Apache personalizzati con percorsi diversi, allinea il mapping.

Sintomo tipico di mapping sbagliato: log dice “Connected…”, ma PhpStorm/VS Code **non si ferma**.

---

## 7) Trigger vs avvio automatico

- **`xdebug.start_with_request = yes`**: più semplice per iniziare; ogni richiesta prova a collegarsi.
- **`trigger`**: attiva solo con `?XDEBUG_TRIGGER=1`, header o cookie (estensione **Xdebug helper**, impostata su **Xdebug v3**).  
  Se usi trigger e “non si ferma”, quasi sempre **manca il trigger**.

---

## 8) Verificare il PHP “giusto” (multi-install)

Spesso su Windows ci sono più PHP (Herd, XAMPP, vecchi zip). Verifica il binario usato:
```powershell
where php
php -v
php --ini
```
In VS Code, controlla `"php.validate.executablePath"`. In PhpStorm, verifica l’**Interpreter**/CLI path. In Docker, usa `sail exec ... php --ri xdebug` per non sbagliare SAPI.

---

## 9) Script e rotte di prova

### 9.1 Rotta Laravel “_debug”
```php
// routes/web.php
Route::get('/_debug', function () {
    $x = ['hello' => 'world', 't' => time()];
    return 'ok';
});
```
Metti un break in questa closure: è il punto più affidabile per il primo test.

### 9.2 Script CLI minimo (per VS Code “Launch current file”)
```php
<?php
// test.php
foreach (range(0, 3) as $i) {
    $x = $i * 2; // breakpoint qui
    echo "i=$i, x=$x\n";
}
```
Usa la config **Launch currently open script (CLI)**.

---

## 10) Firewall, antivirus, VPN

- Consenti alla tua IDE connessioni in **ingresso** (porta 9003).
- Alcune VPN possono filtrare loopback/porte locali. Prova a disattivarle temporaneamente.
- Su Windows, verifica regole del **Windows Defender Firewall**.

---

## 11) Problemi tipici e fix rapidi

- **“Xdebug not found”**  
  DLL sbagliata (TS/NTS, x86/x64) o ini errato (FPM vs CLI). Soluzione: **Wizard** Xdebug + modifica nel **php.ini della CLI**; conferma con `php --ri xdebug`.

- **“Could not connect to debugging client”**  
  IDE non ascolta o porta errata/occupata, firewall/VPN, host non raggiungibile. Accendi listener, libera 9003 o cambia porta in modo coerente, controlla `client_host`.

- **Break non scatta ma connected nel log**  
  Path mapping errato (Docker) o break su riga non eseguita. Metti break su route d’ingresso e fissa i mapping.

- **PhpStorm ignora la sessione**  
  “Ignore connections from unknown servers” attivo senza Server definito. Definisci Server e `PHP_IDE_CONFIG` corrispondente.

- **VS Code non esegue il PHP giusto**  
  Correggi `"php.validate.executablePath"`, riapri terminale, verifica `php -v`.

---

## 12) Checklist finale operativa

- IDE in ascolto su **9003** (una sola IDE alla volta).
- `xdebug.mode=develop,debug`, `start_with_request=yes|trigger`.
- Host coerente: `127.0.0.1` in locale; `host.docker.internal`/`172.17.0.1` in Docker.
- PhpStorm: Server definito + `PHP_IDE_CONFIG` (Docker o “unknown servers” attivo).
- Docker/Sail: path mappings corretti `/var/www/html` ↔ root progetto.
- Rotta di prova `/_debug` o script CLI minimo per validare la catena.
- Log Xdebug solo per diagnosi e poi **disattivalo**.

Se questa checklist passa, il debug con Xdebug 3 è **affidabile** sia in locale che in container.
