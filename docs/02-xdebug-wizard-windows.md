# Xdebug con Wizard (Windows)
1. Incolla `php -i` su xdebug.org/wizard, scarica la DLL consigliata.
2. Copia la DLL in `ext`, abilita in `php.ini`:
   - `zend_extension = xdebug`
   - `xdebug.mode = develop,debug`
   - `xdebug.start_with_request = yes`
   - `xdebug.client_port = 9003`
   - `xdebug.client_host = 127.0.0.1`
3. Verifica: `php --ri xdebug`. Migliora i var_dump con i limiti di visualizzazione.
