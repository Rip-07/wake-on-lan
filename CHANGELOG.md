# Changelog

## 0.1.0 — 2026-11-01

First public release.

- HTTP service on Axum: `GET /health`, `GET /computers`, `POST /computers/{id}/wake`.
- Magic Packet sent to the computer's IPv4 broadcast address, UDP port 9.
- Computers stored in PostgreSQL via SeaORM; the `computers` table is created on startup.
- API key in the `X-Api-key` header, compared in constant time (SHA-256 + `subtle`).
- Startup fails with a clear error when `API_KEY` or `DATABASE_URL` is missing.
- Release build with LTO; Docker image running as a non-root user; CI with fmt, clippy, tests and image build.
