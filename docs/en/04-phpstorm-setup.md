# PhpStorm: Server, mapping, PHP_IDE_CONFIG
- Debug port 9003; enable listener.
- Local Server: Host 127.0.0.1, Port 8000 (artisan serve). No path mappings locally.
- With Docker/Sail: map `/var/www/html` and set `PHP_IDE_CONFIG=serverName=<Name>`.
