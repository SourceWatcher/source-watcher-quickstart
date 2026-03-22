# Source Watcher

PHP ETL (Extract-Transform-Load) with a REST API and web UI. Three projects:

| Project | Role |
|---------|------|
| **[source-watcher-core](https://github.com/TheCocoTeam/source-watcher-core)** | ETL engine (extractors, transformers, loaders) |
| **[source-watcher-api](https://github.com/TheCocoTeam/source-watcher-api)** | REST API (auth, DB, endpoints) |
| **[source-watcher-board](https://github.com/TheCocoTeam/source-watcher-board)** | Web UI (login, transformations canvas) |

**Stack:** Core and API require **PHP 8.4**. The board is a front-end that calls the API over HTTP.

---

## Folder structure

This repository (`source-watcher-dev-env`) is the development environment. It expects the following layout, with all three sub-projects cloned as siblings inside this folder:

```
source-watcher-dev-env/
  source-watcher-core/    ← clone of source-watcher-core
  source-watcher-api/     ← clone of source-watcher-api
  source-watcher-board/   ← clone of source-watcher-board
  docker-compose.dev.yml
  Dockerfile.dev
  README.md
```

Clone all three before running any commands:

```bash
git clone https://github.com/TheCocoTeam/source-watcher-core
git clone https://github.com/TheCocoTeam/source-watcher-api
git clone https://github.com/TheCocoTeam/source-watcher-board
```

---

## Commands (from `source-watcher-dev-env/` root)

### 1. Build the Docker dev environment (first time)

Do this once, and again after any change to `Dockerfile.dev`:

```bash
docker compose -f docker-compose.dev.yml build
```

If Compose fails, build the image directly:

```bash
docker build -f Dockerfile.dev -t source-watcher-dev:latest .
```

### 2. Install dependencies

Install **Core** first, then **API** (the API depends on Core via a Composer path repository).

**Core:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "composer install --no-interaction --ignore-platform-reqs"
```

**API:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "composer install --no-interaction --ignore-platform-reqs"
```

### 3. Update lock files

**Core:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "composer update --no-interaction --ignore-platform-reqs"
```

**API:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "composer update --no-interaction --ignore-platform-reqs"
```

### 4. Run tests

Run after installing dependencies (step 2).

**Core:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "./vendor/bin/phpunit"
```

**API:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "./vendor/bin/phpunit"
```

### 5. Run code coverage

Requires the dev image (step 1). Run after installing dependencies (step 2).

**Core:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core source-watcher-dev:latest sh -c "./vendor/bin/phpunit --coverage-text"
```

Open `source-watcher-core/phpunit-report/coverage-html/index.html` in a browser for a line-by-line HTML report.

**API:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api source-watcher-dev:latest sh -c "./vendor/bin/phpunit --coverage-text"
```

---

## Running the stack

### 6. Run the API

From inside `source-watcher-dev-env/source-watcher-api/`:

```bash
cd source-watcher-api
docker compose up -d --build api
```

- **API:** http://localhost:8181/
- MySQL runs in the same stack on port 3306.

To stop:
```bash
docker compose down
```

**Environment variables:** Copy `.env.example` to `.env` inside `source-watcher-api/` and adjust if needed. The defaults work for local development.

**Default login credentials** are seeded by `UserSeeder.php` (`src/phinx/Database/Seeds/UserSeeder.php`). Check that file for the current username and password. The default password is `secret`.

### 7. Run the board

Start the API first (step 6), then from inside `source-watcher-dev-env/source-watcher-board/`:

```bash
cd source-watcher-board
docker compose up -d --build web-server
```

- **Board:** http://localhost:8080/
- Log in with the credentials from step 6.

To stop:
```bash
docker compose down
```

**API URL configuration:** The board's API base URL (`http://localhost:8181`) is configured directly in `html/transformations.php`. Update the `STEPS_API_URL`, `TRANSFORMATION_API_URL`, and `TRANSFORMATION_RUN_API_URL` constants near the top of that file if your API runs on a different host or port.

---

## Pipeline data directory

The API container mounts `.source-watcher/` from `source-watcher-api/` into `/var/www/html/.source-watcher/` inside the container. This directory holds:

```
source-watcher-api/.source-watcher/
  transformations/    ← .json pipeline files (loaded by the board and API)
  data/               ← local input files (CSVs, text files, etc.)
  *.db                ← SQLite output files written by pipeline runs
```

Example pipelines are available at **[source-watcher-examples](https://github.com/TheCocoTeam/source-watcher-examples)**. To use them, clone or copy the `.json` pipeline files into `source-watcher-api/.source-watcher/transformations/`.

---

## Samples (source-watcher-core)

Runnable ETL pipelines live under **source-watcher-core/samples/**. See the [source-watcher-core samples README](https://github.com/TheCocoTeam/source-watcher-core/tree/master/samples#readme) for how to run each sample.
