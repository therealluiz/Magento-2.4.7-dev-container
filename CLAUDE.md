# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Docker Compose-based local development environment for Magento 2.4.7. The repository contains Docker configuration and helper scripts only — the actual Magento source code is installed into `src/` (gitignored) during setup.

**Stack:** PHP 8.3, Nginx 1.24, MySQL 8.0, OpenSearch 2.12, Redis 7.2, RabbitMQ 3.13, MailHog

## Setup and Configuration

1. Copy `.env.example` to `.env` and set `COMPOSER_AUTH` (Magento Marketplace credentials are required)
2. Run `bin/setup` for automated first-time installation (creates project, runs `setup:install`, configures all services)

All `bin/` scripts wrap `docker compose exec` calls — they must be run from the repo root with containers running.

## Common Commands

```bash
# Container lifecycle
bin/start                    # Start all services
bin/stop                     # Stop all services
bin/restart [service]        # Restart one or all services
bin/status                   # Show container status
bin/destroy [--volumes]      # Remove containers (optionally with volumes)

# Shell access
bin/cli                      # Shell as 'magento' user
bin/root                     # Shell as root

# Magento CLI
bin/magento <command>        # Run bin/magento inside container
bin/magerun <command>        # Run n98-magerun2

# PHP tooling
bin/composer <args>          # Composer commands
bin/phpunit <args>           # PHPUnit (tests in src/)
bin/phpcs <args>             # PHP CodeSniffer
bin/phpcbf <args>            # PHP Code Beautifier & Fixer

# Frontend
bin/node / bin/npm / bin/grunt

# Magento workflow
bin/cache [flush|clean|status|enable|disable]
bin/reindex [status|reset|info]
bin/deploy                   # Deploy static content
bin/di-compile               # DI compilation
bin/upgrade                  # setup:upgrade
bin/dev-mode [developer|production]
bin/module [status|enable|disable] <ModuleName>
bin/permissions              # Fix file ownership

# Debugging
bin/xdebug [on|off|profile|trace|coverage]   # Modifies .env + signals container
bin/spx [on|off]             # Toggle SPX profiler (web UI at /spx/)

# Database
bin/mysql [-e "SQL"]         # Interactive or one-liner
bin/db-import <file>         # Import SQL dump
bin/db-export                # Export database

# Redis
bin/redis-flush [cache|fpc|sessions|all]

# Logs
bin/logs [service|magento]   # Tail logs (use 'magento' for var/log/)
```

## Architecture

### Repository Structure

```
bin/             # 47 shell scripts — all wrappers around docker compose exec
docker/
  php/           # Dockerfile, php.ini, xdebug.ini, spx.ini
  nginx/         # default.conf (Magento-specific config)
  mysql/         # my.cnf
docker-compose.yml
.env.example
src/             # Magento source (gitignored, created by bin/setup)
```

### Docker Services

| Service | Image | Port |
|---------|-------|------|
| php | custom (php:8.3-fpm-bookworm) | 9000 (internal) |
| nginx | nginx:1.24-alpine | 80 |
| db | mysql:8.0 | 3306 |
| opensearch | opensearchproject/opensearch:2.12.0 | 9200 |
| redis | redis:7.2-alpine | 6379 |
| rabbitmq | rabbitmq:3.13-management-alpine | 5672, 15672 |
| mailhog | mailhog/mailhog | 8025, 1025 |

### PHP Container

- User `magento` (UID/GID from `.env`) owns all files in `src/` — always run Magento commands as this user (the `bin/` scripts handle this)
- Xdebug 3.3 pre-installed but **off by default** (port 9003, IDE key: PHPSTORM)
- SPX profiler available at `http://localhost/spx/` with key `dev`
- `bin/xdebug on/off` dynamically changes `.env` and sends SIGUSR2 to reload without restart

### Redis Cache Layout

| DB | Purpose |
|----|---------|
| 0 | Cache |
| 1 | Full-page cache |
| 2 | Sessions |

### Magento Source Location

All Magento development happens inside `src/`. Custom modules go in `src/app/code/`. Run Magento CLI commands via `bin/magento` or `bin/cli` to get a shell.

## Default Credentials

| Service | User | Password |
|---------|------|----------|
| Magento Admin (`/admin`) | admin | admin123 |
| MySQL | magento | magento |
| RabbitMQ Management UI | magento | magento |

## Environment Variables

Key `.env` variables that affect development:

- `XDEBUG_MODE` — `off`, `debug`, `profile`, `trace`, `coverage`
- `SPX_ENABLED` / `SPX_KEY` — SPX profiler toggle and access key
- `INSTALL_SAMPLE_DATA` — Set to `1` before `bin/setup` to include sample data
- `COMPOSER_AUTH` — JSON with Magento Marketplace credentials (required)
