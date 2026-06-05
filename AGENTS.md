# Dograh - Project Overview

Dograh is a voice AI platform for building and deploying conversational AI agents with telephony and WebRTC support.

## Project Structure

```
dograh/
├── api/              # Backend - FastAPI application
├── ui/               # Frontend - Next.js application
├── scripts/          # Helper scripts for local development
├── docs/             # Mintlify documentation
├── pipecat/          # Pipecat framework (git submodule)
├── docker-compose.yaml       # Production/OSS deployment
├── docker-compose-local.yaml # Local development services
```

## Tech Stack

- **Backend**: Python with FastAPI
- **Frontend**: Next.js 15 with React 19, TypeScript, Tailwind CSS
- **Database**: PostgreSQL with SQLAlchemy (async)
- **Cache/Queue**: Redis with ARQ for background tasks
- **Storage**: MinIO (S3-compatible) for audio files

## Local Development

Contributor setup and service startup are documented in `docs/contribution/setup.mdx`.

## Environment Configuration

- `api/.env` - Backend environment variables. Source this when running diagnostic scripts or one-off services against the dev DB (e.g. `python -m api.services.admin_utils.local_exec`).
- `api/.env.test` - Test-only environment variables. Source this when running pytest so tests hit the test DB and never the dev/prod credentials in `api/.env`.
- `ui/.env` - Frontend environment variables

Typical invocation:

```bash
# Tests
source venv/bin/activate && set -a && source api/.env.test && set +a && python -m pytest api/tests/...

# Diagnostics / scripts
source venv/bin/activate && set -a && source api/.env && set +a && python -m api.services.admin_utils.local_exec
```

## Common Commands

Development is **devcontainer-first** (see `docs/contribution/setup.mdx`); run these inside the container unless noted. Toolchain: Python 3.13, Node 22. On a Windows host (outside the container), the `.sh` scripts below have `.ps1` equivalents where a pair exists (`setup_requirements`, `start_services_dev`, `stop_services`, `makemigrate`, `migrate` — see `scripts/AGENTS.md`); `lint`/`format`/`pre_commit` are bash-only.

### Setup

```bash
./scripts/setup_requirements.sh --dev   # install api + pipecat Python deps
(cd ui && npm install)                  # install frontend deps
```

### Run locally

```bash
bash scripts/start_services_dev.sh   # backend: uvicorn api.app:app --reload on :8000 + single arq worker, waits for /api/v1/health
(cd ui && npm run dev)               # frontend on :3000 (add `-- --hostname 0.0.0.0` inside the container)
curl localhost:8000/api/v1/health    # verify the backend is up
```

### Tests (backend, pytest)

Source `api/.env.test` first so tests hit the test DB, never dev/prod (see *Environment Configuration*). Config lives in `api/pytest.ini` (`asyncio_mode = auto`).

```bash
python -m pytest api/tests/                                # full suite
python -m pytest api/tests/path/test_file.py              # one file
python -m pytest api/tests/path/test_file.py::test_name   # one test
python -m pytest api/tests/ -m "not slow"                 # skip slow-marked tests
```

### Lint / format / typecheck

```bash
bash scripts/lint.sh        # Python: mypy + ruff check + ruff format --check
bash scripts/format.sh      # autofix: ruff (api + pipecat) + ui eslint --fix
bash scripts/pre_commit.sh  # runs format.sh, then re-stages the files it fixed
(cd ui && npm run lint)     # frontend eslint
(cd ui && npm run build)    # frontend production build (also typechecks)
```

### Codegen & migrations

```bash
(cd ui && npm run generate-client)     # regenerate ui/src/client/ from backend OpenAPI after adding/changing a route
./scripts/makemigrate.sh "description" # create an Alembic migration from model changes
./scripts/migrate.sh                   # apply migrations
```
