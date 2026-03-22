# Source Watcher

PHP ETL (Extract-Transform-Load) with a REST API and web UI.

| Project | Role |
|---------|------|
| **[source-watcher-core](https://github.com/SourceWatcher/source-watcher-core)** | ETL engine (extractors, transformers, loaders) |
| **[source-watcher-api](https://github.com/SourceWatcher/source-watcher-api)** | REST API (auth, DB, pipeline endpoints) |
| **[source-watcher-board](https://github.com/SourceWatcher/source-watcher-board)** | Web UI (login, drag-and-drop pipeline canvas) |

**Stack:** PHP 8.4, Docker. No local PHP needed - everything runs in containers.

---

## Get running

### Step 1 - Clone this repo and the three sub-projects

```bash
git clone https://github.com/SourceWatcher/source-watcher-quickstart
cd source-watcher-quickstart

git clone https://github.com/SourceWatcher/source-watcher-core
git clone https://github.com/SourceWatcher/source-watcher-api
git clone https://github.com/SourceWatcher/source-watcher-board
```

Your folder should look like this:

```
source-watcher-quickstart/
  source-watcher-core/
  source-watcher-api/
  source-watcher-board/
  Dockerfile.dev
  docker-compose.dev.yml
  README.md
```

### Step 2 - Set up the API environment

```bash
cd source-watcher-api
cp .env.example .env
cd ..
```

The defaults in `.env` work for local development. No changes needed to get started.

### Step 3 - Install Composer dependencies

Run these from inside `source-watcher-quickstart/` (the root of this repo):

**Core** (install first - the API depends on it):
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "composer install --no-interaction --ignore-platform-reqs"
```

**API:**
```bash
docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "composer install --no-interaction --ignore-platform-reqs"
```

### Step 4 - Start the API

From inside `source-watcher-quickstart/` (the root of this repo):

```bash
cd source-watcher-api
docker compose up -d --build api
cd ..
```

- **API:** http://localhost:8181/

To stop: `docker compose down` (from inside `source-watcher-api/`)

### Step 5 - Start the board

From inside `source-watcher-quickstart/` (the root of this repo):

```bash
cd source-watcher-board
docker compose up -d --build web-server
cd ..
```

- **Board:** http://localhost:8080/

To stop: `docker compose down` (from inside `source-watcher-board/`)

### Step 6 - Log in

Open http://localhost:8080/ and log in with the default credentials seeded by `source-watcher-api/src/phinx/Database/Seeds/UserSeeder.php`:

- **Username:** `jpruiz114`
- **Password:** `secret`

---

## Pipeline data directory

The API container mounts `source-watcher-api/.source-watcher/` as `/var/www/html/.source-watcher/` inside the container:

```
source-watcher-api/.source-watcher/
  transformations/    ← .json pipeline files (loaded by the board and API)
  data/               ← local input files (CSVs, text files, images, etc.)
  *.db                ← SQLite output files written by pipeline runs
```

Example pipelines are available at **[source-watcher-examples](https://github.com/SourceWatcher/source-watcher-examples)**. Clone or copy the `.json` files into `source-watcher-api/.source-watcher/transformations/` to use them.

---

## Development

These commands are for contributors working on the core or API. Not needed just to run the stack.

### Build the dev image

Do this once, and again after any change to `Dockerfile.dev`:

```bash
docker compose -f docker-compose.dev.yml build
```

If Compose fails, build the image directly:

```bash
docker build -f Dockerfile.dev -t source-watcher-dev:latest .
```

### Update Composer lock files

Run from inside `source-watcher-quickstart/` (the root of this repo):

**Core:**
```bash
docker compose -f docker-compose.dev.yml run --rm php sh -c \
  "cd source-watcher-core && composer update --no-interaction --ignore-platform-reqs"
```

**API:**
```bash
docker compose -f docker-compose.dev.yml run --rm php sh -c \
  "cd source-watcher-api && composer update --no-interaction --ignore-platform-reqs"
```

### Run tests

**Core:**
```bash
docker compose -f docker-compose.dev.yml run --rm php sh -c \
  "cd source-watcher-core && ./vendor/bin/phpunit"
```

**Core — with terminal coverage summary:**
```bash
docker compose -f docker-compose.dev.yml run --rm php sh -c \
  "cd source-watcher-core && ./vendor/bin/phpunit --coverage-text"
```

The HTML report is always written to `source-watcher-core/phpunit-report/coverage-html/index.html` automatically. Open it in a browser after any test run.

**Core — verbose (shows each test name and any skipped/risky tests):**
```bash
docker compose -f docker-compose.dev.yml run --rm php sh -c \
  "cd source-watcher-core && ./vendor/bin/phpunit --coverage-text --verbose"
```

### Board API URL

The board's API base URL is configured in `source-watcher-board/html/transformations.php` via the `STEPS_API_URL`, `TRANSFORMATION_API_URL`, and `TRANSFORMATION_RUN_API_URL` constants near the top of that file. The default (`http://localhost:8181`) works for local development.

### Core samples

Runnable ETL pipelines live under `source-watcher-core/samples/`. See the [samples README](https://github.com/SourceWatcher/source-watcher-core/tree/master/samples#readme) for details.
