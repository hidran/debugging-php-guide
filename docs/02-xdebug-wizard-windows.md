# Xdebug con Wizard su Windows (Herd) – Installazione e Configurazione Pro

Questa guida spiega **come installare e configurare Xdebug 3 su Windows** quando PHP è stato installato con **Herd**. Userai il **Wizard ufficiale** di Xdebug per scaricare la **DLL corretta**, la copierai nella cartella **ext** della tua versione di PHP e configurerai il `php.ini` **della CLI** (quello usato da `php artisan serve`). Concludiamo con suggerimenti per **var_dump leggibili**, modalità di avvio **`start_with_request`**, uso del **trigger**, e una sezione di **troubleshooting**.

---

## 1) Verifica la versione e l’INI della tua CLI

Prima di tutto assicurati di lavorare sulla **CLI** (Command Line Interface), non su FPM/Apache:

```powershell
php -v
php --ini
where php
```

- `php --ini` ti mostra dove sono caricati i file di configurazione della **CLI** (es. `C:\Users\<UTENTE>\AppData\Roaming\Herd\bin\php\8.x\php.ini` o `…\conf.d\`).
- **Annota** il percorso: è lì che andremo ad abilitare Xdebug.

> Se `where php` non punta a Herd, rivedi PATH (pagina 01).

---

## 2) Usa il Wizard ufficiale per la DLL corretta

1. Esegui:
   ```powershell
   php -i > phpinfo.txt
   ```
   Questo crea un file con tutte le info del tuo PHP (architettura, TS/NTS, API).

2. Apri **https://xdebug.org/wizard** e incolla **l’intero contenuto** di `phpinfo.txt`.

3. Il Wizard ti dirà **esattamente** quale **DLL** scaricare (es.: `php_xdebug-3.x.x-8.3-vs16-nts-x64.dll`) e **dove copiarla**.

4. Scarica la DLL e **rinominala** in `php_xdebug.dll` (facoltativo, ma rende più semplice la riga di `zend_extension`).

---

## 3) Copia la DLL nella cartella `ext` della tua versione PHP

- Percorso tipico con Herd (esempio):
  ```
  C:\Users\<UTENTE>\AppData\Roaming\Herd\bin\php\8.x\ext\
  ```
- Copia lì `php_xdebug.dll`.

> Se non hai la cartella `ext`, crea la directory e posiziona la DLL indicata dal Wizard.

---

## 4) Abilita Xdebug nel `php.ini` **della CLI**

Apri **l’`ini` della CLI** che hai annotato al §1 e aggiungi **una sola volta** le righe Xdebug (evita duplicati):

```ini
; === Xdebug – abilitazione modulo ===
zend_extension = php_xdebug.dll

; === Modalità consigliate in sviluppo ===
xdebug.mode = develop,debug
xdebug.start_with_request = yes

; === Porta e host del client (IDE) ===
xdebug.client_port = 9003
xdebug.client_host = 127.0.0.1

; === (Opzionale) Log diagnostico ===
; xdebug.log = C:\Temp\xdebug.log
; xdebug.log_level = 7
```

**Note importanti**
- `zend_extension`: se non hai rinominato la DLL, usa il nome esatto fornito dal Wizard (es. `zend_extension="C:\...\ext\php_xdebug-3.3.1-...dll"`). Evita virgolette se non necessarie e **nessuno spazio** attorno a `=`.
- `client_port`: Xdebug 3 usa **9003** di default (non 9000).
- `client_host`: per debug locale è `127.0.0.1`.

Salva il file e **riapri il terminale** (così PHP rilegge l’ini).

---

## 5) Verifica che Xdebug sia caricato

```powershell
php --ri xdebug
```

Dovresti vedere:
- **Version** di Xdebug;
- `xdebug.mode => develop,debug`;
- `xdebug.start_with_request => yes` (se hai scelto avvio automatico);
- `xdebug.client_port => 9003`.

Se ottieni “Xdebug not found”, ricontrolla **percorso DLL**, `zend_extension`, e di essere nell’ini **CLI** corretto.

---

## 6) Scegli come avviare il debugger

### Opzione A – Sempre attivo (facile durante il setup)
```ini
xdebug.start_with_request = yes
```
Ogni richiesta CLI/web (gestita dalla CLI via `artisan serve`) proverà a collegarsi all’IDE.

### Opzione B – Solo su richiesta (trigger)
```ini
xdebug.start_with_request = trigger
```
Attivi Xdebug **solo** quando aggiungi il **trigger**:
- Query string: `?XDEBUG_TRIGGER=1`
- Header: `XDEBUG_TRIGGER: 1`
- Cookie via estensione browser **Xdebug helper** (impostata su **Xdebug v3**)

> Il trigger è utile se vuoi performance massime quando **non** stai debuggando.

---

## 7) Migliora i `var_dump` (leggibilità)

Xdebug rende i `var_dump` più belli in `mode=develop`. Puoi controllare i limiti:

```ini
; Limiti di visualizzazione per var_dump
xdebug.var_display_max_children = 256
xdebug.var_display_max_data     = 1024
xdebug.var_display_max_depth    = 5
```

---

## 8) Collegare l’IDE (VS Code / PhpStorm)

### VS Code
- Installa estensione **PHP Debug** (xdebug.php-debug).
- Crea `.vscode/launch.json` con una config “**Listen for Xdebug**” su **port 9003**.
- Per debuggare web con `php artisan serve`, avvia:
  ```powershell
  php artisan serve --host=127.0.0.1 --port=8000
  ```
  Poi apri `http://127.0.0.1:8000/...` con il listener **attivo**.

### PhpStorm
- **Settings → PHP → Debug**: porta **9003**; attiva il **telefono** (listening).
- **Senza Docker**: non servono path mappings. Crea un **Server** `127.0.0.1:8000` se usi “Ignore unknown servers”.

> Se **un’altra IDE** è in ascolto sulla stessa porta, potrebbe “rubare” la connessione. Mantieni **una sola** IDE in ascolto o usa porte diverse.

---

## 9) Debug rapido di prova

1. In un progetto Laravel, aggiungi una rotta:
   ```php
   // routes/web.php
   Route::get('/_debug', function () {
       $a = ['hello' => 'world'];
       return 'ok';
   });
   ```
2. Metti un **breakpoint** sul `return`.
3. Avvia:
   ```powershell
   php artisan serve --host=127.0.0.1 --port=8000
   ```
4. Assicurati che l’IDE **ascolti** su 9003 e apri `http://127.0.0.1:8000/_debug`.

Con `start_with_request = yes` **non** serve trigger. Con `trigger`, usa `?XDEBUG_TRIGGER=1`.

---

## 10) Troubleshooting (checklist)

**A) “Xdebug not found” (o non appare in `php --ri xdebug`)**
- `zend_extension` errato (nome DLL o percorso sbagliati).
- DLL scaricata **TS/NTS** o **x86/x64** sbagliata: usa **Wizard** e riscarica quella giusta.
- Stai modificando l’**ini sbagliato** (FPM/Apache invece della **CLI**). Controlla `php --ini`.

**B) “Could not connect to debugging client” nel log**
- IDE **non in ascolto** su 9003 (accendi listener).
- **Altra IDE** sta già usando 9003: chiudila o cambia porta.
- `client_host` non raggiungibile: in debug locale usa `127.0.0.1`.

**C) Il breakpoint non si ferma**
- Con `trigger`: hai messo `?XDEBUG_TRIGGER=1` (o cookie/header)?
- La riga col breakpoint **viene eseguita** davvero?
- In PhpStorm: se “Ignore unknown servers” è **ON**, crea un Server `127.0.0.1:8000` o disattiva l’opzione.

**D) Var_dump ancora “brutti”**
- Controlla `xdebug.mode` includa `develop`.
- Aggiungi i limiti di visualizzazione (§7).

**E) Più PHP installati**
- `where php` deve puntare a Herd (o alla versione scelta). Allinealo dal PATH o imposta l’eseguibile in IDE.

---

## 11) Riepilogo rapido

- Usa il **Wizard** per la **DLL corretta**.
- Metti la DLL in `ext`, abilita con **`zend_extension`** nel **`php.ini` della CLI**.
- Imposta `xdebug.mode=develop,debug` e scegli `start_with_request=yes` (o `trigger`).
- IDE in ascolto su **9003**; una sola IDE alla volta.
- Con `artisan serve` non servono path mappings; usa `127.0.0.1:8000`.

---

**Prossima pagina:** [VS Code – Extension Pack, PHP executable path, launch.json](03-vscode-setup.md)
