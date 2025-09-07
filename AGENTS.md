# Repository Guidelines

## Project Structure & Module Organization
- `_worker.js`: Single-file Cloudflare Worker (entrypoint) that serves the homepage UI and proxies GitHub files and Docker registry (v2) requests. Handles auth challenges and S3 redirects.
- `README.md`: Usage instructions and overview.
- `LICENSE`: Project license.

## Build, Test, and Development Commands
- Local dev (Wrangler): `wrangler dev` after adding a `wrangler.toml` with `main = "_worker.js"`. Runs the Worker locally.
- Deploy (Wrangler): `wrangler deploy` to publish to Cloudflare.
- Quick smoke tests (after deploy or dev):
  - Home: `curl -I https://<your-worker-domain>/`
  - GitHub proxy: `curl -I https://<your-worker-domain>/github.com/<user>/<repo>/releases/download/<tag>/<asset>`
  - Docker v2: `curl -I https://<your-worker-domain>/v2/library/hello-world/manifests/latest`

## Coding Style & Naming Conventions
- JavaScript (Workers runtime): 2-space indentation, semicolons required, prefer single quotes; template literals for HTML.
- Constants: `UPPER_SNAKE_CASE` (e.g., `ALLOWED_HOSTS`, `RESTRICT_PATHS`).
- Functions/vars: `camelCase` (e.g., `handleRequest`, `isAmazonS3`).
- Optional formatting: `npx prettier -w _worker.js`.

## Testing Guidelines
- No formal test suite in this repo; rely on curl-based smoke tests and Wrangler preview.
- Verify headers for Docker paths: response includes `Docker-Distribution-API-Version` and CORS headers.
- Exercise redirects and auth: test 401 → token flow and S3 302/307 handling on `ghcr.io`/`registry-1.docker.io` paths.

## Commit & Pull Request Guidelines
- Commits: short, present-tense messages; scope first when helpful (e.g., `proxy: handle S3 307` or `ui: dark mode input fix`).
- PRs: include description, rationale, and test evidence (curl commands + outputs). Link related issues. Update `README.md` when changing behavior or endpoints.

## Security & Configuration Tips
- Whitelist domains in `ALLOWED_HOSTS`; avoid broadening unnecessarily.
- If stricter control is needed, set `RESTRICT_PATHS = true` and enumerate `ALLOWED_PATHS`.
- Log carefully; avoid leaking tokens. The Worker already adds required AWS headers for S3; do not forward unnecessary `x-amz-*` from clients.

## Architecture Overview
- Single Worker export (`fetch`) dispatches UI (`/`) and proxy logic.
- Normalizes Docker domains, handles Bearer token challenges, and follows redirects while staying in-proxy.
