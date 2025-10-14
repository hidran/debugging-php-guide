# Installare PHP su Windows con Herd (Laravel) – Guida completa

Questa pagina ti guida all’installazione di **PHP su Windows** usando **Herd** (l’installer raccomandato dall’ecosistema Laravel). Al termine avrai PHP funzionante, aggiunto al **PATH**, **Composer** pronto e la possibilità di gestire **più versioni di PHP** in parallelo.

---

## 1) Perché usare Herd

**Herd** è un gestore di runtime PHP che:
- installa **più versioni di PHP** (8.1, 8.2, 8.3, …);
- include **estensioni** pronte (curl, mbstring, intl, openssl, ecc.);
- fornisce un **launcher** (`herd php`, `herd composer`) e può configurare il **PATH**.

Rispetto a zip manuali o stack come XAMPP/WAMP, Herd è più **manutenibile**, integra bene con **Laravel** e semplifica il cambio di **versione PHP** per progetto.

---

## 2) Prerequisiti

- Windows 10/11 aggiornato.
- Permessi di amministratore per installare software.
- Connessione Internet.
- (Consigliato) Terminale **PowerShell**.

---

## 3) Installazione di Herd

1. Scarica l’installer da **https://herd.laravel.com**.
2. Esegui l’installer e segui i passaggi.
3. Apri **Herd** e vai alla sezione **PHP**.

> Suggerimento: tieni chiusi terminali/IDE durante l’installazione; riaprili a fine setup per ereditare le variabili d’ambiente aggiornate.

---

## 4) Installare PHP con Herd

1. In **Herd → PHP**, scegli la versione (es. 8.3/8.2) e clicca **Install**.
2. Imposta la versione **Default** (stella o menu contestuale).
3. Verifica dal terminale:
    - `php -v` → mostra la versione corretta;
    - `herd php -v` → forza l’uso del PHP gestito da Herd nel terminale corrente.

Se `php -v` non riflette la versione installata, vedi §7 (PATH).

---

## 5) Installare e verificare Composer

- **Composer** è incluso: prova `composer -V`.
- In alternativa, usa `herd composer -V` per invocarlo tramite Herd.

Se `composer` non è trovato, vedi §7 per il PATH oppure usa sempre il prefisso `herd`.

---

## 6) Gestire più versioni di PHP

- **Default globale**: scegli in Herd quale versione è la predefinita.
- **Per singolo comando**: usa il selettore versione del launcher:
    - `herd php@8.3 -v`
    - `herd php@8.2 -S 127.0.0.1:8000 -t public`
- **Per progetto (IDE)**: indica all’IDE il **percorso eseguibile** della versione desiderata (vedi pagine dedicate a VS Code/PhpStorm).

---

## 7) Aggiungere PHP al PATH (se necessario)

Se `php`/`composer` non vengono trovati:

1. Apri **Pannello di controllo → Sistema → Impostazioni di sistema avanzate → Variabili d’ambiente**.
2. In **Variabili di sistema** → seleziona **Path** → **Modifica**.
3. Aggiungi il percorso bin di Herd (es. `C:\Users\<UTENTE>\AppData\Roaming\Herd\bin`) **oppure** la cartella della versione PHP mostrata in Herd.
4. Chiudi e **riapri** il terminale, quindi verifica:
    - `where php`
    - `php -v`
    - `composer -V`

> Mantieni il PATH **pulito**: rimuovi voci obsolete (XAMPP/WAMP) per evitare conflitti.

---

## 8) Individuare e modificare `php.ini` (CLI)

- In Herd, ogni versione ha la propria cartella (es. `…\Herd\bin\php\8.x\`).
- Se c’è solo `php.ini-development` o `php.ini-production`, copiane uno in `php.ini` e modificalo.
- Impostazioni utili in sviluppo:
    - `display_errors = On`
    - `error_reporting = E_ALL`
    - `date.timezone = "Europe/Rome"`
    - Estensioni: `extension=curl`, `extension=mbstring`, `extension=intl`, `extension=fileinfo`, `extension=openssl`

> **Nota:** il debugging con `php artisan serve` usa la **CLI**, quindi Xdebug va abilitato nel `php.ini` della **CLI**. La pagina successiva spiega l’abilitazione con il **Wizard di Xdebug**.

---

## 9) Verifiche finali

- `php -v` → versione corretta?
- `where php` → puntamento a Herd?
- `composer -V` → Composer disponibile?
- `php -m` → estensioni necessarie (intl, mbstring…) risultano caricate?

Se tutto ok, PHP è pronto per i capitoli successivi.

---

## 10) Problemi frequenti & soluzioni

**A) `php` non è riconosciuto**
- PATH non aggiornato → aggiungi la cartella corretta (vedi §7) e riapri il terminale.

**B) Versione PHP “sbagliata”**
- Ci sono più PHP nel PATH (XAMPP/WAMP/vecchi zip). Mantieni solo Herd o mettilo prima nella lista. In IDE imposta l’eseguibile esplicito.

**C) Estensioni mancanti**
- Abilita in `php.ini` rimuovendo `;` e verifica con `php -m`.

**D) Composer non trovato**
- Usa `herd composer` o aggiungi al PATH come per `php`.

