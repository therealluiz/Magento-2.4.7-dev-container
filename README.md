# Magento 2.4.7 Dev Container

Fast local development environment for Magento 2.4.7 using Docker Compose.

## Stack

| Service | Version | Purpose |
|---------|---------|---------|
| PHP-FPM | 8.3 | Application runtime |
| Nginx | 1.24 | Web server |
| MySQL | 8.0 | Database |
| OpenSearch | 2.12 | Search engine |
| Redis | 7.2 | Cache + Sessions |
| RabbitMQ | 3.13 | Message queue |
| MailHog | latest | Email catcher |

## Debugging Tools

| Tool | Description |
|------|-------------|
| **Xdebug 3.3** | Step debugging, profiling, tracing, code coverage |
| **SPX Profiler** | Free profiler with built-in web UI (Blackfire alternative) |
| **n98-magerun2** | Magento CLI power tool |

## Prerequisites

- Docker & Docker Compose v2
- [Magento Marketplace keys](https://marketplace.magento.com/customer/accessKeys/)

## Quick Start

```bash
# 1. Clone this repo
git clone <repo-url> && cd Magento-2.4.7-dev-container

# 2. Configure environment
cp .env.example .env
# Edit .env — set your COMPOSER_AUTH with Marketplace keys

# 3. Start services & install Magento
bin/start
bin/setup
```

Magento will be available at **http://localhost** (admin: http://localhost/admin).

## Commands

### General

| Command | Description |
|---------|-------------|
| `bin/start` | Start all containers |
| `bin/stop` | Stop all containers |
| `bin/restart` | Restart all (or `bin/restart php` for one service) |
| `bin/status` | Show status of all containers |
| `bin/setup` | First-time Magento installation |
| `bin/destroy` | Remove containers (`--volumes` to wipe data) |
| `bin/logs` | Tail all logs (`bin/logs php`, `bin/logs magento`) |
| `bin/cli` | Shell into PHP container as magento user |
| `bin/root` | Root shell (`bin/root php`, `bin/root db`, etc.) |

### PHP Container

| Command | Description |
|---------|-------------|
| `bin/php` | Run PHP commands |
| `bin/composer` | Run Composer commands |
| `bin/magento` | Run `bin/magento` commands |
| `bin/magerun` | Run n98-magerun2 commands |
| `bin/node` | Run Node.js commands |
| `bin/npm` | Run npm commands |
| `bin/grunt` | Run Grunt (Magento frontend workflow) |
| `bin/phpunit` | Run PHPUnit tests |
| `bin/phpcs` | Run PHP CodeSniffer |
| `bin/phpcbf` | Run PHP Code Beautifier & Fixer |

### Magento Workflow

| Command | Description |
|---------|-------------|
| `bin/cache` | Flush cache (`flush`, `clean`, `status`, `enable`, `disable`) |
| `bin/reindex` | Reindex all (`status`, `reset`, `info`) |
| `bin/cron` | Run cron (`install`, `remove`, `group <name>`) |
| `bin/deploy` | Deploy static content |
| `bin/di-compile` | Run DI compilation |
| `bin/upgrade` | Run `setup:upgrade` |
| `bin/dev-mode` | Show/switch deploy mode (`developer`, `production`) |
| `bin/module` | Module management (`status`, `enable`, `disable`) |
| `bin/permissions` | Fix file/directory permissions |

### Debugging

| Command | Description |
|---------|-------------|
| `bin/xdebug on` | Enable step debugging (port 9003) |
| `bin/xdebug off` | Disable Xdebug |
| `bin/xdebug profile` | Enable Xdebug profiling (cachegrind) |
| `bin/xdebug trace` | Enable function tracing |
| `bin/xdebug coverage` | Enable code coverage |
| `bin/spx on` | Enable SPX profiler web UI |
| `bin/spx off` | Disable SPX profiler |

### MySQL

| Command | Description |
|---------|-------------|
| `bin/mysql` | MySQL client (interactive or `bin/mysql -e "QUERY"`) |
| `bin/mysqldump` | Dump database to stdout |
| `bin/mysqladmin` | Run mysqladmin commands (`status`, `processlist`) |
| `bin/db-import` | Import SQL file (`bin/db-import backup.sql` or `.sql.gz`) |
| `bin/db-export` | Export DB to timestamped file (`--no-gzip` option) |

### Redis

| Command | Description |
|---------|-------------|
| `bin/redis` | redis-cli (`ping`, `info`, `dbsize`, `keys`, etc.) |
| `bin/redis-flush` | Flush Redis (`cache`, `fpc`, `sessions`, `all`) |
| `bin/redis-monitor` | Real-time Redis command monitor |

### OpenSearch

| Command | Description |
|---------|-------------|
| `bin/opensearch` | Query REST API (`_cat/indices?v`, etc.) |
| `bin/opensearch-health` | Show cluster health + index summary |
| `bin/opensearch-indices` | List indices (optional pattern filter) |

### RabbitMQ

| Command | Description |
|---------|-------------|
| `bin/rabbitmq` | Run rabbitmqctl commands |
| `bin/rabbitmq-queues` | List queues with message counts |
| `bin/rabbitmq-purge` | Purge queue (`<name>` or `--all`) |

### Nginx

| Command | Description |
|---------|-------------|
| `bin/nginx` | Run nginx commands (`-t`, `-T`, `-s reload`) |
| `bin/nginx-reload` | Test and reload Nginx config |
| `bin/nginx-logs` | Tail logs (`access`, `error`, or both) |

## Xdebug Setup

Xdebug 3.3 is pre-installed but **disabled by default** (zero performance overhead).

```bash
# Enable step debugging
bin/xdebug on

# Enable profiling (generates cachegrind files)
bin/xdebug profile
```

**PHPStorm**: Set debug port to `9003`, server name to `magento`, path mapping `/var/www/html` -> `./src`.

**VS Code**: Install PHP Debug extension, add launch config with port `9003` and pathMappings.

## SPX Profiler

[SPX](https://github.com/NoiseByNorthwest/php-spx) is a free, self-hosted profiler with a built-in web UI — no external accounts needed.

```bash
# Enable SPX
bin/spx on

# Access the web UI
open "http://localhost/?SPX_KEY=dev&SPX_UI_URI=/"

# Profile a specific page
# Just add ?SPX_KEY=dev&SPX_ENABLED=1 to any URL
```

Features: flamegraphs, timeline view, 22 metrics (time, memory, I/O, objects), call stack analysis.

## Service URLs

| Service | URL |
|---------|-----|
| Storefront | http://localhost |
| Admin Panel | http://localhost/admin |
| RabbitMQ Management | http://localhost:15672 |
| MailHog | http://localhost:8025 |
| OpenSearch | http://localhost:9200 |

## Performance Optimizations

- MySQL tmpfs for temp tables + relaxed durability
- Redis for all cache/session storage (no filesystem cache)
- OPcache dev-tuned (revalidate every request)
- Composer cache persisted in Docker volume
- Xdebug off by default (no overhead until enabled)
- JIT disabled (incompatible with Xdebug, minimal benefit in dev)

## Default Credentials

| Service | Username | Password |
|---------|----------|----------|
| Magento Admin | admin | admin123 |
| MySQL | magento | magento |
| RabbitMQ | magento | magento |
