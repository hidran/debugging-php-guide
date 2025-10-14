# VS Code – Setup completo: PHP Extension Pack, PHP executable path, launch.json

Questa pagina spiega come configurare **Visual Studio Code** per sviluppare e fare **debug** in PHP su **Windows**, dopo aver installato PHP con **Herd** e abilitato **Xdebug** (vedi Pagina 02). Al termine avrai **IntelliSense** (auto-completamento), **linting**, e un **debugger** funzionante sia per **script CLI** sia per **richieste web** (con `php artisan serve`).

> **Prerequisiti**
> - `php -v`, `where php` → PHP raggiungibile (quello di Herd)
> - `php --ri xdebug` → Xdebug 3 attivo in **CLI** (Version, mode, port 9003)
> - Un progetto PHP/Laravel aperto in VS Code (cartella root)

---

## 1) Installa le estensioni giuste

Apri **Extensions** (Ctrl+Shift+X) e installa:
- **PHP Intelephense** (IntelliSense avanzato)
- **PHP Debug** (id: `xdebug.php-debug`) ← **indispensabile** per il debug
- (Facoltativi ma utili) **EditorConfig**, **Error Lens**, **DotENV**

Riavvia VS Code se richiesto.

---

## 2) Imposta il **PHP executable path**

VS Code deve usare **lo stesso PHP** della CLI configurata (Herd).

**Via Settings GUI**
1. `Ctrl+,` → cerca **php validate executable path**
2. Imposta il percorso, ad esempio:
   ```
   C:\Users\<UTENTE>\AppData\Roaming\Herd\bin\php\php.exe
   ```

**Oppure via file** `.vscode/settings.json` (crealo se non esiste):
```json
{
  "php.validate.enable": true,
  "php.validate.executablePath": "C:\\Users\\<UTENTE>\\AppData\\Roaming\\Herd\\bin\\php\\php.exe",
  "intelephense.environment.phpVersion": "8.3",
  "editor.formatOnSave": true
}
```

**Verifica in terminale integrato (View → Terminal):**
```powershell
php -v
```
Deve corrispondere al PHP di Herd. Se vedi un’altra build, correggi il path.

---

## 3) Crea `.vscode/launch.json`

Questo file definisce come VS Code interagisce con Xdebug. Useremo due configurazioni “base”:

- **Listen for Xdebug (Web)** → ascolta connessioni DBGp (per `artisan serve`)
- **Launch currently open script (CLI)** → esegue e debuggga il file aperto

Crea/edita `.vscode/launch.json` nella **root** del progetto:

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

**Note chiave**
- **Porta**: Xdebug 3 usa **9003** (verifica che non sia occupata da altre IDE).
- “Web” **ascolta**; “CLI” **esegue** un file con Xdebug attivo.
- Se vuoi personalizzare, puoi aggiungere altre config (profiling, server integrato, ecc.).

---

## 4) Debug **Web** con `php artisan serve`

1. Avvia il server Laravel dal terminale:
   ```powershell
   php artisan serve --host=127.0.0.1 --port=8000
   ```
2. In VS Code → **Run & Debug** → seleziona **Listen for Xdebug (Web)** → **Start**.
3. Metti un **breakpoint** in un punto sicuro, ad es. `routes/web.php`:
   ```php
   Route::get('/_debug', function () {
       $x = ['hello' => 'world'];
       return 'ok';
   });
   ```
4. Apri il browser su `http://127.0.0.1:8000/_debug`.

> Se nel `php.ini` hai `xdebug.start_with_request = trigger`, aggiungi `?XDEBUG_TRIGGER=1` all’URL o usa l’estensione **Xdebug helper** (impostata su Xdebug v3). Con `yes` non serve alcun trigger.

---

## 5) Debug **CLI** (file singolo, comandi)

Per debuggare rapidamente `test.php` o piccoli script:
1. Apri il file in VS Code.
2. Metti un **breakpoint**.
3. Avvia **Launch currently open script (CLI)**.

Questa config lancia `php` con `XDEBUG_MODE=debug,develop` e `-dxdebug.start_with_request=yes`, collegandosi alla porta 9003 del tuo VS Code.

**Esempi pratici**
- Script: `test.php`, job singoli, snippet.
- Comandi artisan personalizzati (puoi puntare `program` a `artisan` e passare args in `args`).

---

## 6) Alternative utili

### 6.1 Debug dei test (Pest/PHPUnit)
Esegui dal terminale integrato:
```powershell
$env:XDEBUG_MODE="debug"
php -dxdebug.start_with_request=yes artisan test
```
I breakpoint in codice **testato** scatteranno (VS Code deve avere il listener attivo o usare una config CLI dedicata).

### 6.2 Server integrato PHP (progetti non-Laravel)
Aggiungi una terza config:
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

### 6.3 Profiling (prestazioni)
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
Analizza i file generati con (Q)Cachegrind.

---

## 7) Breakpoint & tips

- **Raggiungibilità**: piazza i primi breakpoint in punti certi (route, controller d’ingresso).
- **Conditional breakpoint**: click destro sul breakpoint → condizioni/espressioni utili per filtri.
- **Logpoints**: stampano messaggi in Debug Console senza fermare l’esecuzione.
- **Step into/over/out**: impara le scorciatoie (F11/F10/Shift+F11) per fluire velocemente.

---

## 8) Errori comuni (pitfalls)

- **IDE concorrente**: se PhpStorm è in ascolto su 9003, “ruba” la connessione → chiudilo o cambia porta.
- **PHP “sbagliato”**: in VS Code il terminale mostra un PHP diverso (non Herd) → correggi `"php.validate.executablePath"` e riapri il terminale.
- **Xdebug non caricato in CLI**: `php --ri xdebug` deve mostrare la versione e `mode` attivo.
- **Host/URL** incoerenti: usa **127.0.0.1** (non `localhost`) e mantieni coerenza nelle config.
- **Trigger mancante**: in modalità trigger, ricordati `?XDEBUG_TRIGGER=1` o cookie/header.
- **Breakpoint “grigi”**: spesso il file aperto non è quello eseguito, o la riga non viene mai toccata.

---

## 9) Troubleshooting rapido

- **Non si ferma**
    - Listener VS Code **attivo** su 9003?
    - `xdebug.start_with_request = yes` (o trigger usato)?
    - La riga è davvero eseguita?
- **Log Xdebug**
    - Attiva temporaneamente nel `php.ini` CLI:
      ```ini
      xdebug.log = C:\Temp\xdebug.log
      xdebug.log_level = 7
      ```
    - Se vedi “**Connected to debugging client**” ma nessun break → è un problema di raggiungibilità del breakpoint o config lato IDE.
    - “**Could not connect to debugging client**” → l’IDE non ascolta/porta errata/porta occupata.

Disattiva il log quando hai finito (scrive molto).

---

## 10) Riepilogo

- Installa **PHP Debug** + **Intelephense**.
- Imposta il **PHP executable path** verso Herd.
- Crea `launch.json` con “Listen for Xdebug (Web)” e “Launch currently open script (CLI)”.
- Per Laravel: avvia `php artisan serve` su `127.0.0.1:8000`, ascolta su 9003, piazza breakpoint su route/controller.
- Risolvi conflitti di porta/IDE e usa il log Xdebug per diagnosi mirate.

---

**Prossima pagina:** PhpStorm – Server, mapping, `PHP_IDE_CONFIG`
