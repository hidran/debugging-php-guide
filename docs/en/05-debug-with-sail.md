# Debug with Laravel Sail (Docker)
- Add services (MariaDB, Valkey, Mailpit).
- Enable Xdebug: `SAIL_XDEBUG_MODE=develop,debug`.
- macOS/Win: client_host=host.docker.internal — Linux: 172.17.0.1.
- PhpStorm: Server + mapping; VS Code: pathMappings.
