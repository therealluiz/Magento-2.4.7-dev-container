# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Docker Compose-based local development environment for Magento 2.4.7. The repository contains Docker configuration and helper scripts only — the actual Magento source code is installed into `src/` (gitignored) during setup.

**Stack:** PHP 8.3, Nginx 1.24, MySQL 8.0, OpenSearch 2.12, Redis 7.2, RabbitMQ 3.13, MailHog

## Setup and Configuration

1. Copy `.env.example` to `.env` and set `COMPOSER_AUTH` (Magento Marketplace credentials are required)
2. Run `bin/setup` for automated first-time installation (creates project, runs `setup:install`, configures all services)

`COMPOSER_AUTH` must be a JSON string with your Magento Marketplace keys:
```json
{"http-basic":{"repo.magento.com":{"username":"PUBLIC_KEY","password":"PRIVATE_KEY"}}}
```

All `bin/` scripts wrap `docker compose exec` calls — they must be run from the repo root with containers running.

## Common Commands

```bash
# Container lifecycle
bin/start                    # Start all services (auto-creates .env if missing)
bin/stop                     # Stop all services
bin/restart [service]        # Restart one or all services
bin/status                   # Show container status and port mappings
bin/destroy [--volumes]      # Remove containers (optionally with volumes)
bin/setup                    # First-time automated Magento installation

# Shell access
bin/cli                      # Shell as 'magento' user inside PHP container
bin/root                     # Shell as root inside PHP container
bin/php <args>               # Execute raw PHP commands

# Magento CLI
bin/magento <command>        # Run bin/magento inside container
bin/magerun <command>        # Run n98-magerun2

# PHP tooling
bin/composer <args>          # Composer commands
bin/phpunit <args>           # PHPUnit (tests in src/)
bin/phpcs <args>             # PHP CodeSniffer
bin/phpcbf <args>            # PHP Code Beautifier & Fixer

# Frontend
bin/node <args>              # Node.js commands
bin/npm <args>               # npm package manager
bin/grunt <args>             # Grunt for Magento frontend workflow

# Magento workflow
bin/cache [flush|clean|status|enable|disable]
bin/reindex [status|reset|info]
bin/cron                     # Manage Magento cron jobs
bin/deploy                   # Deploy static content
bin/di-compile               # DI compilation
bin/upgrade                  # setup:upgrade
bin/dev-mode [developer|production]
bin/module [status|enable|disable] <ModuleName>
bin/permissions              # Fix file ownership

# Debugging & Profiling
bin/xdebug [on|off|profile|trace|coverage]   # Modifies .env + signals container
bin/spx [on|off]             # Toggle SPX profiler (web UI at /spx/)

# Database
bin/mysql [-e "SQL"]         # Interactive MySQL or one-liner
bin/mysqldump                # Dump entire database to stdout
bin/mysqladmin <args>        # Run mysqladmin commands (status, processlist, etc.)
bin/db-import <file>         # Import SQL dump (.sql or .sql.gz)
bin/db-export                # Export database to timestamped file

# Redis
bin/redis                    # Interactive redis-cli
bin/redis-flush [cache|fpc|sessions|all]
bin/redis-monitor            # Real-time Redis command monitoring

# OpenSearch
bin/opensearch               # Query REST API with curl wrapper
bin/opensearch-health        # Display cluster health and index summary
bin/opensearch-indices       # List indices with optional pattern filtering

# RabbitMQ
bin/rabbitmq <args>          # Run rabbitmqctl commands
bin/rabbitmq-queues          # List queues with message counts
bin/rabbitmq-purge           # Purge specific queue or all queues

# Nginx
bin/nginx <args>             # Run nginx commands (test, reload, etc.)
bin/nginx-reload             # Test and reload Nginx configuration
bin/nginx-logs               # Tail Nginx access/error logs

# Logs
bin/logs [service|magento]   # Tail logs (use 'magento' for var/log/)
```

## Architecture

### Repository Structure

```
bin/             # 47 shell scripts — all wrappers around docker compose exec
docker/
  php/           # Dockerfile, php.ini, xdebug.ini, spx.ini
  nginx/         # default.conf (Magento-specific Nginx config)
  mysql/         # my.cnf (performance tuning)
docker-compose.yml
.env.example
src/             # Magento source (gitignored, created by bin/setup)
```

### Docker Services

| Service | Container Name | Image | Port(s) |
|---------|---------------|-------|---------|
| php | magento-php | custom (php:8.3-fpm-bookworm) | 9000 (internal) |
| nginx | magento-nginx | nginx:1.24-alpine | 80 |
| db | magento-db | mysql:8.0 | 3306 |
| opensearch | magento-opensearch | opensearchproject/opensearch:2.12.0 | 9200 |
| redis | magento-redis | redis:7.2-alpine | 6379 |
| rabbitmq | magento-rabbitmq | rabbitmq:3.13-management-alpine | 5672, 15672 |
| mailhog | magento-mailhog | mailhog/mailhog | 8025, 1025 |

All services communicate over a single custom bridge network named `magento`.

### PHP Container Details

- User `magento` (UID/GID from `.env`, default 1000:1000) owns all files in `src/` — always run Magento commands as this user (the `bin/` scripts handle this automatically)
- Xdebug 3.3 pre-installed but **off by default** (port 9003, IDE key: PHPSTORM)
- SPX profiler available at `http://localhost/spx/` with key `dev`
- `bin/xdebug on/off` dynamically changes `.env` and sends SIGUSR2 to PHP-FPM (no container restart needed)
- Composer 2.7, Node.js 18, and n98-magerun2 are pre-installed

**PHP Extensions:** bcmath, ctype, curl, dom, fileinfo, filter, gd (with freetype/jpeg/webp), hash, iconv, intl, json, libxml, mbstring, opcache, openssl, pdo_mysql, simplexml, soap, sockets, sodium, tokenizer, xmlwriter, xsl, zip, zlib

**Key PHP settings (`docker/php/php.ini`):**
- `memory_limit = 4G`
- `max_execution_time = 1800`
- `opcache.revalidate_freq = 0` (checks files on every request in dev mode)
- `opcache.jit = off` (incompatible with Xdebug)
- Mail routed through MailHog via `sendmail_path`

### MySQL Configuration (`docker/mysql/my.cnf`)

Performance-tuned for development:
- `innodb_buffer_pool_size = 512M`
- `innodb_flush_log_at_trx_commit = 0` (speed over durability — dev only)
- `max_allowed_packet = 64M`
- `tmp_table_size = 64M` + 512M tmpfs `/tmp` for temp tables
- `character-set-server = utf8mb4` / `collation-server = utf8mb4_unicode_ci`

### Nginx Configuration (`docker/nginx/default.conf`)

- FastCGI upstream: `php:9000` with 600s timeout and 756MB buffer
- Document root: `/var/www/html/pub`
- Gzip enabled for all text/JS/CSS/SVG content
- Static files cached with 1-year expiration, version strings stripped (`/static/version{N}/` → `/static/`)
- SPX profiler routed at `/spx/`
- Security: blocks `.git`, `.htaccess`, raw `.php` access, sensitive media paths
- Header: `X-Frame-Options: SAMEORIGIN`

### Redis Cache Layout

| DB | Purpose |
|----|---------|
| 0 | Cache |
| 1 | Full-page cache (FPC) |
| 2 | Sessions |

### Docker Volumes

| Volume | Purpose |
|--------|---------|
| `db-data` | MySQL persistent data |
| `opensearch-data` | OpenSearch persistent data |
| `composer-cache` | Persistent Composer cache (speeds up rebuilds) |

### Magento Source Location

All Magento development happens inside `src/`. Custom modules go in `src/app/code/`. Run Magento CLI commands via `bin/magento` or use `bin/cli` to get a shell.

## bin/setup — What It Does

The automated installer performs these steps in order:
1. Validates `.env` and `COMPOSER_AUTH` presence
2. Starts all services and waits for healthchecks (db, redis, opensearch)
3. Creates Magento 2.4.7 project via `composer create-project` from `repo.magento.com`
4. Runs `setup:install` with all service connections pre-configured
5. Sets developer deploy mode
6. Disables 2FA modules for dev convenience
7. Runs all indexers
8. Flushes cache
9. Optionally installs sample data (if `INSTALL_SAMPLE_DATA=1`)

## Default Credentials

| Service | URL | User | Password |
|---------|-----|------|----------|
| Magento Admin | `http://localhost/admin` | admin | admin123 |
| MySQL | localhost:3306 | magento | magento |
| RabbitMQ Management UI | `http://localhost:15672` | magento | magento |
| MailHog Web UI | `http://localhost:8025` | — | — |
| SPX Profiler | `http://localhost/spx/` | key: `dev` | — |

MySQL root password is also `magento`.

## Environment Variables

All variables come from `.env` (copy from `.env.example`):

| Variable | Default | Description |
|----------|---------|-------------|
| `USER_ID` / `GROUP_ID` | 1000 / 1000 | Maps host user into container for file ownership |
| `COMPOSER_AUTH` | _(empty)_ | JSON with Magento Marketplace keys — **required** |
| `XDEBUG_MODE` | `off` | `off`, `debug`, `profile`, `trace`, `coverage` |
| `SPX_ENABLED` | `0` | Set to `1` to enable SPX profiler |
| `SPX_KEY` | `dev` | SPX web UI access key |
| `INSTALL_SAMPLE_DATA` | `0` | Set to `1` before `bin/setup` to include sample data |
| `NGINX_PORT` | `80` | Host port for Nginx |
| `MYSQL_PORT` | `3306` | Host port for MySQL |
| `OPENSEARCH_PORT` | `9200` | Host port for OpenSearch |
| `REDIS_PORT` | `6379` | Host port for Redis |
| `RABBITMQ_PORT` | `5672` | Host port for RabbitMQ AMQP |
| `RABBITMQ_MGMT_PORT` | `15672` | Host port for RabbitMQ Management UI |
| `MAILHOG_PORT` | `8025` | Host port for MailHog Web UI |

## Xdebug Usage

```bash
bin/xdebug on        # Step debugging (port 9003, IDE key: PHPSTORM)
bin/xdebug off       # Disable completely
bin/xdebug profile   # Generate cachegrind.out.* files in /tmp
bin/xdebug trace     # Function trace
bin/xdebug coverage  # Code coverage (for PHPUnit)
```

`bin/xdebug` modifies `XDEBUG_MODE` in `.env` and sends SIGUSR2 to PHP-FPM — no container restart required. The `xdebug.client_host = host.docker.internal` setting works on Docker Desktop (Mac/Windows). On Linux, you may need to set this to your host IP.

## SPX Profiler Usage

```bash
bin/spx on    # Enable SPX web UI
bin/spx off   # Disable SPX
```

Access the profiler at `http://localhost/spx/` using key `dev`. Profiling is triggered manually per-request — SPX auto-start is off by default.

## Magento Development Conventions

- **Never run Magento CLI commands as root** — always use `bin/magento` or `bin/cli` (which runs as the `magento` user)
- **After adding/modifying modules:** run `bin/upgrade` then `bin/di-compile` then `bin/cache flush`
- **After changing templates/static files:** run `bin/deploy` or use developer mode (where static files generate on demand)
- **Custom modules** go in `src/app/code/<Vendor>/<Module>/`
- **Composer packages** go in `src/vendor/` (managed by `bin/composer`)
- **File permissions** can be reset with `bin/permissions` if they get corrupted

## Typical Development Workflow

```bash
# First time
cp .env.example .env
# Edit .env: set COMPOSER_AUTH
bin/setup

# Daily
bin/start
bin/magento cache:flush
# ... develop ...
bin/stop

# After module changes
bin/upgrade && bin/di-compile && bin/cache flush

# Debug a request
bin/xdebug on
# ... set breakpoint in IDE, make request ...
bin/xdebug off

# Profile a slow page
bin/spx on
# ... visit /spx/ to trigger and view profile ...
bin/spx off

# Check logs
bin/logs magento         # Magento var/log/
bin/logs nginx           # Nginx access/error
bin/logs php             # PHP-FPM logs
```
