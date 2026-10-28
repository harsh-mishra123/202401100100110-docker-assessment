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

## Stage 4: Version and publish

- Built with `--build-arg APP_VERSION=1.0.0` and tagged `harrsshhh/fest-backend:1.0.0` and `harrsshhh/fest-backend:1.0` (never `latest`), both pushed to the public repo https://hub.docker.com/r/harrsshhh/fest-backend.
- **Digest of 1.0.0:** `sha256:859c79bdb56da6ce5fdd2ee2b8302afd60593daa59313b174a7956032a752ab0` (same digest for `1.0`).
- `compose.yaml` now uses `image: harrsshhh/fest-backend:1.0.0` instead of `build`. After deleting the local backend images, `docker compose up -d` pulled it from Docker Hub and the page shows `backend 1.0.0` with the saved registrations.

## Stage 5: Sign and verify with cosign

- `*.key` is in `.gitignore`. The key pair was generated with `cosign generate-key-pair` and is password-protected. Only `cosign.pub` is committed.
- 1.0.0 is signed by digest, not tag: `cosign sign --key cosign.key harrsshhh/fest-backend@sha256:859c79bdb56da6ce5fdd2ee2b8302afd60593daa59313b174a7956032a752ab0` (Rekor tlog index 3535579971).
- `cosign verify --key cosign.pub harrsshhh/fest-backend:1.0.0` passes.
- 1.0.1 (`APP_VERSION=1.0.1`, digest `sha256:9c7171c7a2c5f44dee5f7eef3dd59758d4dcbbbd3c7a0d5f435779df047d680c`) was pushed **unsigned**. `cosign verify --key cosign.pub harrsshhh/fest-backend:1.0.1` fails with `no signatures found` (exit code 10).
- Docker Content Trust (`docker trust`) was not used.

## Steps not completed

- None.
