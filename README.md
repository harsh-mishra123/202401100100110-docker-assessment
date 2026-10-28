# FestPass — Docker & Container Security (CA-2)

| Item | Value |
|---|---|
| Name | Harsh Mishra |
| REGNO | 202401100100110 |
| PORT | 8010 |
| Docker Hub username | harrsshhh |
| Docker Hub repository | https://hub.docker.com/r/harrsshhh/fest-backend |

## AI tools used

- **Claude Code (Claude Opus 5.5)**: wrote the Dockerfile, `.dockerignore`, `compose.yaml` and this README, explained the Trivy results, and ran the build, scan, push and cosign commands in my terminal on my machine.

## Stack

- `db`: `postgres:17-alpine`, user and database `festpass`, password from `.env`, data in the named volume `festpass-data`, healthcheck with `pg_isready`.
- `backend`: Flask API on port 5000. Starts only once `db` is healthy.
- `frontend`: `nginx:1.30-alpine` on port 8010, proxies `/api/` to `backend:5000`.
- Networks: `frontend-net` (frontend + backend) and `backend-net` (backend + db), so the frontend cannot reach the database.

Run it with `cp .env.example .env` (then set a password) and `docker compose up -d`.

## Stage 3: Trivy before and after

| Image | Base | CRITICAL | HIGH | Size |
|---|---|---|---|---|
| `fest-backend:0.1` | `python:3.9` (Debian 13), root, Flask 2.2.2 | 234 | 2249 | 1.61 GB (412 MB compressed) |
| `fest-backend:0.2` | `python:3.13-alpine3.24`, non-root `appuser`, Flask 3.1.2 | 0 | 0 | 96.5 MB (23.9 MB compressed) |

Fixes: pinned Alpine Python base instead of the full Debian `python:3.9` image, runs as non-root user `appuser` (uid 10001), upgraded Flask 3.1.2, Werkzeug 3.1.3 and psycopg2-binary 2.9.11, and removed pip/setuptools from the runtime image (pip bundles vulnerable copies of urllib3 and msgpack).

`trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2` exits with **0**.

Full reports: [`reports/trivy-before.txt`](reports/trivy-before.txt), [`reports/trivy-after.txt`](reports/trivy-after.txt).
