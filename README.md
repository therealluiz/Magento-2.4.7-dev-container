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

| Command | Description |
|---------|-------------|
| `bin/start` | Start all containers |
| `bin/stop` | Stop all containers |
| `bin/cli` | Shell into PHP container |
| `bin/magento` | Run `bin/magento` commands |
| `bin/composer` | Run Composer commands |
| `bin/xdebug on` | Enable step debugging (port 9003) |
| `bin/xdebug off` | Disable Xdebug |
| `bin/xdebug profile` | Enable Xdebug profiling |
| `bin/spx on` | Enable SPX profiler |
| `bin/spx off` | Disable SPX profiler |

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
