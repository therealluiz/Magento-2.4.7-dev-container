# Magento 2.4.7 Fast Dev Container — Implementation Plan

## System Requirements (from Adobe docs)

| Service | Version |
|---------|---------|
| PHP | 8.3 |
| MySQL | 8.0 |
| Elasticsearch | 8.11 (or OpenSearch 2.12) |
| Redis | 7.2 |
| RabbitMQ | 3.13 |
| Composer | 2.7+ |
| Nginx | 1.24+ |
| Varnish | 7.4 (optional for dev) |
| Node.js | 18+ (for frontend build) |

### Required PHP Extensions
bcmath, ctype, curl, dom, fileinfo, filter, gd, hash, iconv, intl, json, libxml, mbstring, openssl, pdo_mysql, simplexml, soap, sockets, sodium, tokenizer, xmlwriter, xsl, zip, zlib

---

## Architecture: Docker Compose Multi-Service

Optimized for **speed** with:
- Pre-built PHP image with all extensions
- tmpfs for MySQL temp tables
- Redis for session + cache (no file-based caching)
- OpenSearch instead of Elasticsearch (lighter, no license issues)
- Composer cache volume for fast installs
- Mutagen or native bind mounts with delegated consistency

---

## Files to Create

### 1. `docker-compose.yml`
Multi-service orchestration:
- **php** — PHP 8.3-FPM with all Magento extensions, Composer, magerun2
- **nginx** — Reverse proxy with Magento-tuned config
- **db** — MySQL 8.0 with performance-tuned my.cnf
- **opensearch** — OpenSearch 2.12 (single-node dev mode)
- **redis** — Redis 7.2 (cache + sessions)
- **rabbitmq** — RabbitMQ 3.13 with management UI
- **mailhog** — Email catcher for dev

### 2. `docker/php/Dockerfile`
- Base: `php:8.3-fpm-bookworm`
- Install all required PHP extensions via `docker-php-ext-install` and `pecl`
- Install Composer 2.7, magerun2, Node 18
- OPcache tuned for dev (revalidate_freq=0)
- Xdebug 3 pre-installed (disabled by default, toggle via env)
- Non-root user `magento` with proper UID/GID mapping

### 3. `docker/nginx/default.conf`
- Magento 2 Nginx config (from official sample)
- FastCGI pass to php container
- Static file serving optimized

### 4. `docker/mysql/my.cnf`
- InnoDB buffer pool sized for dev (512M)
- tmpdir on tmpfs for speed
- Relaxed durability settings for dev speed (`innodb_flush_log_at_trx_commit=0`)
- `max_allowed_packet=64M`

### 5. `docker/php/php.ini`
- `memory_limit=4G`
- `max_execution_time=1800`
- `realpath_cache_size=10M`
- `realpath_cache_ttl=7200`
- OPcache settings for dev
- Xdebug config (client_host, mode=off by default)

### 6. `docker/php/xdebug.ini`
- Xdebug 3 config, off by default
- Toggle with `XDEBUG_MODE` env var

### 7. `bin/setup` (shell script)
- Automated Magento installation script:
  1. `composer create-project` Magento 2.4.7 (or install from existing source)
  2. `bin/magento setup:install` with all service connection params
  3. Configure Redis for cache + sessions
  4. Configure OpenSearch as search engine
  5. Configure RabbitMQ
  6. Deploy sample data (optional, via env flag)
  7. Set developer mode
  8. Disable 2FA for dev convenience

### 8. `bin/start` / `bin/stop` / `bin/cli`
- Helper scripts:
  - `bin/start` — `docker compose up -d`
  - `bin/stop` — `docker compose down`
  - `bin/cli` — exec into PHP container as magento user
  - `bin/magento` — run bin/magento commands inside container
  - `bin/composer` — run composer inside container
  - `bin/xdebug` — toggle xdebug on/off

### 9. `.env.example`
- All configurable env vars:
  - `MAGENTO_VERSION=2.4.7`
  - `PHP_MEMORY_LIMIT=4G`
  - `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, etc.
  - `XDEBUG_MODE=off`
  - `COMPOSER_AUTH` (for repo.magento.com keys)
  - `INSTALL_SAMPLE_DATA=0`

### 10. `.gitignore`
- Ignore `src/` (Magento source), `.env`, vendor artifacts

### 11. Updated `README.md`
- Quick start guide
- Prerequisites (Docker, Docker Compose)
- Setup instructions
- Available commands
- Accessing services (URLs, ports)
- Xdebug setup for IDEs
- Performance tips

---

## Port Mapping

| Service | Port |
|---------|------|
| Nginx (HTTP) | 80 |
| MySQL | 3306 |
| OpenSearch | 9200 |
| Redis | 6379 |
| RabbitMQ AMQP | 5672 |
| RabbitMQ UI | 15672 |
| Mailhog UI | 8025 |
| Xdebug | 9003 |

---

## Performance Optimizations
1. **MySQL tmpfs** — `/tmp` mounted as tmpfs for temp tables
2. **Redis** for all caching — no filesystem cache
3. **OPcache dev-tuned** — timestamps checked every request
4. **Composer cache volume** — persisted between rebuilds
5. **Delegated mount consistency** — faster file sync on macOS
6. **MySQL relaxed durability** — faster writes in dev
7. **Single-node OpenSearch** — minimal overhead, no cluster
8. **PHP-FPM tuning** — pm.max_children sized for dev workload
