# Source Watcher

PHP ETL (Extract–Transform–Load) with a REST API and web UI. Three projects:

| Project | Role |
|---------|------|
| **source-watcher-core** | ETL engine (extractors, transformers, loaders) |
| **source-watcher-api** | REST API (auth, DB, endpoints) |
| **source-watcher-board** | Web UI (login, transformations canvas) |

**Stack:** Core and API require **PHP 8.4**. The board is a front-end that calls the API over HTTP; see **source-watcher-board/README.md** to run it.

## Commands (from repo root)

Use `docker run` so you don't depend on Docker Compose networks (avoids iptables issues on some hosts).

### 1. Build the Docker dev environment (first time)

Do this once, and again after any change to `Dockerfile.dev`:

```bash
sudo docker compose -f docker-compose.dev.yml build
```

If Compose fails, build the image directly:

```bash
sudo docker build -f Dockerfile.dev -t source-watcher-dev:latest .
```

### 2. Installing dependencies

Install **Core** first, then **API** (the API uses Core via a Composer path repo and needs `source-watcher-core` as a sibling directory).

**Core:**
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "composer install --no-interaction --ignore-platform-reqs"
```

**API:** (from repo root so the path to Core is valid)
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "composer install --no-interaction --ignore-platform-reqs"
```

### 3. Updating lock file

**Core:**
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "composer update --no-interaction --ignore-platform-reqs"
```

**API:** (from repo root so the path to Core is valid)
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "composer update --no-interaction --ignore-platform-reqs"
```

### 4. Running tests

Run after installing dependencies (step 2).

**Core:**
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core composer:2 sh -c "./vendor/bin/phpunit"
```

**API:**
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api composer:2 sh -c "./vendor/bin/phpunit"
```

### 5. Running code coverage

Requires the dev image (step 1). Run after installing dependencies (step 2).

**Core:**
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-core source-watcher-dev:latest sh -c "./vendor/bin/phpunit --coverage-text"
```

- **Text:** summary is printed in the terminal.
- **HTML:** open `source-watcher-core/phpunit-report/coverage-html/index.html` in a browser for a line-by-line report.

**API:**
```bash
sudo docker run --rm -v "$(pwd)":/app -w /app/source-watcher-api source-watcher-dev:latest sh -c "./vendor/bin/phpunit --coverage-text"
```

**Optional:** After building `source-watcher-dev:latest`, you can use it instead of `composer:2` for the commands above and drop `--ignore-platform-reqs`.

### 6. Running the API (Docker)

The API serves auth and endpoints on port **8181** and needs MySQL. From the **dev-env** directory:

```bash
cd source-watcher-api
sudo docker compose up -d --build api
```

- **API:** http://localhost:8181/  
- MySQL runs in the same stack; the board (step 7) should call this API URL for login.

To stop:

```bash
cd source-watcher-api
sudo docker compose down
```

See **source-watcher-api/README.md** for env vars and endpoints.

### 7. Running the board (Docker)

The board is a separate web app (PHP + Apache). Run it with its own Compose file so the API can stay on port 8181 and the board on 8080. Start the **API** first (step 6).

From the **dev-env** directory:

```bash
cd source-watcher-board
sudo docker compose up -d --build web-server
```

- **Board:** http://localhost:8080/
- **phpMyAdmin** (if you use the full board compose): http://localhost:5000/

Ensure the Source Watcher API (e.g. **source-watcher-api** on port 8181) is running and that the board’s API base URL in `html/assets/js/views/login.js` and `html/assets/js/views/transformations.js` matches your API URL.

To stop:

```bash
cd source-watcher-board
sudo docker compose down
```

See **source-watcher-board/README.md** for API URL configuration and project layout.

## Samples (source-watcher-core)

Runnable ETL pipelines live under **source-watcher-core/samples/**. See **source-watcher-core/samples/README.md** for how to run each sample (Docker commands).
