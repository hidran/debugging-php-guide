# Xdebug via Wizard (Windows)
1. Paste `php -i` into https://xdebug.org/wizard.
2. Download the suggested DLL, copy it into `ext`.
3. Enable in `php.ini`:
   - `zend_extension = xdebug`
   - `xdebug.mode = develop,debug`
   - `xdebug.start_with_request = yes`
   - `xdebug.client_port = 9003`
   - `xdebug.client_host = 127.0.0.1`
4. Confirm with `php --ri xdebug`. Optional: prettier var_dump limits.
