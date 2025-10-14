# Debug con `php artisan serve` (senza Docker): setup rapido e troubleshooting

Questa pagina spiega come fare **debug** di un progetto Laravel **senza Docker**, usando il server integrato (`php artisan serve`). Il vantaggio è la **semplicità**: niente path mappings, meno variabili in gioco. Lato Xdebug, però, devi assicurarti che sia attivo nella **CLI** (Command Line Interface), non in FPM/Apache.

> Prerequisiti:
> - PHP installato localmente (consigliato con **Herd** su Windows)
> - Xdebug 3 attivo per **CLI** (`php --ri xdebug` mostra versione/mode)
> - IDE: **PhpStorm** o **VS Code** con listener su **porta 9003**

---

## 1) Abilitare Xdebug in CLI

Trova dove la **CLI** carica i file ini e abilita Xdebug lì:

```powershell
php --ini
```

Apri l’ini relativo (es. `C:\Users\<UTENTE>\AppData\Roaming\Herd\bin\php\8.x\php.ini`) e assicurati che ci sia **una sola** abilitazione Xdebug:

```ini
zend_extension = php_xdebug.dll
xdebug.mode = develop,debug
xdebug.start_with_request = yes    ; oppure: trigger
xdebug.client_port = 9003
xdebug.client_host = 127.0.0.1
; xdebug.log = C:\Temp\xdebug.log
; xdebug.log_level = 7
```

> In alternativa, puoi evitare di toccare l’ini e attivare “al volo” quando avvii il server:
> ```powershell
> $env:XDEBUG_MODE="develop,debug"
> php -dxdebug.start_with_request=yes artisan serve --host=127.0.0.1 --port=8000
> ```

---

## 2) Avvio del server

Comando consigliato (host esplicito, porta nota):

```powershell
php artisan serve --host=127.0.0.1 --port=8000
```

> Specificare **127.0.0.1** evita mismatch con `localhost` in alcune configurazioni.

---

## 3) PhpStorm (senza Docker)

1. **Settings → PHP → Debug**: porta **9003**, **Start Listening** attivo.
2. **Settings → PHP → Servers** → **+**
    - **Name**: `LocalLaravel`
    - **Host**: `127.0.0.1`
    - **Port**: `8000`
    - **Debugger**: Xdebug
    - **Use path mappings**: **OFF**
3. Se tieni attivo **“Ignore connections from unknown servers”**:
    - Esporta prima di avviare il server:
      ```powershell
      $env:PHP_IDE_CONFIG="serverName=LocalLaravel"
      ```
    - Il nome deve **combaciare** con **Name** del Server.

Metti un breakpoint in `routes/web.php` o in un controller e apri `http://127.0.0.1:8000/_debug`.

---

## 4) VS Code (senza Docker)

1. Installa **PHP Debug** (xdebug.php-debug).
2. Crea `.vscode/launch.json` con:
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
3. Avvia `php artisan serve`, seleziona **Listen for Xdebug (Web)** e metti un breakpoint.
4. Apri `http://127.0.0.1:8000/_debug`.
    - Con `trigger`: usa `?XDEBUG_TRIGGER=1` o l’estensione **Xdebug helper**.

---

## 5) Debug dei test / comandi CLI

- Test (Pest/PHPUnit):
  ```powershell
  $env:XDEBUG_MODE="debug"
  php -dxdebug.start_with_request=yes artisan test
  ```
- Comandi personalizzati:
  ```powershell
  $env:XDEBUG_MODE="debug"
  php -dxdebug.start_with_request=yes artisan my:command
  ```

I breakpoint scatteranno nel codice eseguito (controller/service ecc.). Ricorda: qui **non** serve alcun mapping.

---

## 6) Troubleshooting mirato

**A) Nessun break**
- IDE **non in ascolto** su 9003 (attiva listener)
- `start_with_request` non è `yes` e non stai usando trigger
- La riga del breakpoint **non** viene eseguita (mettilo su una route di prova)
- Con `localhost` vs `127.0.0.1` a volte non corrisponde l’host → uniforma

**B) “Could not connect to debugging client” nel log**
- Nessuna IDE in ascolto o porta errata/occupata
- Altra IDE (PhpStorm/VS Code) sta **rubando** la connessione

**C) Xdebug non caricato in CLI**
- `php --ri xdebug` deve mostrare la **Version** e `mode`
- Stai editando l’ini sbagliato (FPM/Apache invece della CLI)

**D) Breakpoint “grigio” in PhpStorm**
- Con “Ignore connections from unknown servers” attivo: definisci il Server o esporta `PHP_IDE_CONFIG`

**E) Performance scarse**
- Usa `start_with_request=trigger` e abilita il trigger solo quando serve

---

## 7) Riepilogo

- Abilita Xdebug nella **CLI** (ini giusto).
- Avvia `artisan serve` su **127.0.0.1:8000**.
- PhpStorm/VS Code in ascolto su **9003**.
- Nessun path mapping richiesto.
- Usa `trigger` se vuoi attivare il debug **solo quando serve**.

---

**Prossima pagina:** Troubleshooting avanzato (log Xdebug, porta, host, server mapping, doppie IDE, script di verifica).
