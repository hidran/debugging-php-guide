# PhpStorm: Server, mapping, PHP_IDE_CONFIG
- Debug port: 9003; attiva il listener.
- Server locale: Host 127.0.0.1, Port 8000 (artisan serve). Nessun path mapping in locale.
- Con Docker/Sail: mappa `/var/www/html`, e passa `PHP_IDE_CONFIG=serverName=<NomeServer>`.
