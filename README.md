# wake-on-lan

[![CI](https://github.com/Rip-07/wake-on-lan/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Rip-07/wake-on-lan/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Rust 2024](https://img.shields.io/badge/rust-edition%202024-orange.svg)](Cargo.toml)
[![Image](https://img.shields.io/badge/image-ghcr.io%2Frip--07%2Fwake--on--lan-2496ED.svg)](https://github.com/Rip-07/wake-on-lan/pkgs/container/wake-on-lan)

Wake up computers on your LAN with one authenticated HTTP call: a small Rust service (Axum + SeaORM + PostgreSQL) that stores your machines and sends them a Magic Packet. Built for a homelab (Unraid, Docker with host networking).

[Русская версия ниже](#русский)

## Features

- `GET /health` — public liveness check.
- `GET /computers` — list registered computers (API key required).
- `POST /computers/{id}/wake` — send a Magic Packet to the computer's broadcast address, UDP port 9 (API key required).
- API key is checked in constant time (SHA-256 digests compared with `subtle`).
- The `computers` table is created automatically on startup.
- Release build with LTO, small non-root Docker image.

## Configuration

| Variable | Required | Description |
|---|---|---|
| `API_KEY` | yes | Token clients send in the `X-Api-key` header. Must not be empty. |
| `DATABASE_URL` | yes | PostgreSQL URL, e.g. `postgres://user:pass@host:5432/db`. |
| `PORT` | no | HTTP port, default `8084` (`HOST_PORT` in `docker-compose.yml`). |

The service exits with a clear error if `API_KEY` or `DATABASE_URL` is missing. See `.env.example`.

## Quick start

Requires Linux with Docker (host networking; on Docker Desktop for Mac/Windows UDP broadcast does not reach your LAN).

```bash
git clone https://github.com/Rip-07/wake-on-lan && cd wake-on-lan
cp .env.example .env                      # set API_KEY (openssl rand -hex 32)
docker compose --profile db up -d --build # service + bundled PostgreSQL
```

Prebuilt image: `ghcr.io/rip-07/wake-on-lan` (replace `build: .` with `image: ghcr.io/rip-07/wake-on-lan:0.1.0`).

Already have PostgreSQL? Point `DATABASE_URL` at it and run `docker compose up -d --build` without the profile.

Host networking (`network_mode: host`) is required: UDP broadcast does not leave a bridged Docker network.

From source:

```bash
export API_KEY=... DATABASE_URL=postgres://...
cargo run --release
```

## Add a computer

```bash
docker compose exec db psql -U wol -c \
  "INSERT INTO computers (name, mac_address, broadcast_address) VALUES ('Workstation', 'AA:BB:CC:DD:EE:FF', '192.0.2.255');"
```

MAC may use `:` or `-` separators. `broadcast_address` is the IPv4 broadcast address of the target subnet.

## API

```bash
curl http://127.0.0.1:8084/health
curl -H "X-Api-key: $API_KEY" http://127.0.0.1:8084/computers
curl -X POST -H "X-Api-key: $API_KEY" http://127.0.0.1:8084/computers/1/wake
```

`wake` returns `202 Accepted` with `{"message": "...", "computer_name": "..."}`; `404` for an unknown id, `422` for an invalid MAC/broadcast address in the database, `401` without a valid key.

## Development

```bash
cargo fmt --all --check
cargo clippy --all-targets --all-features --locked -- -D warnings
cargo test --locked
```

CI (GitHub Actions) runs the same checks plus `docker build`.

## License

MIT — see [LICENSE](LICENSE).

---

## Русский

HTTP-сервис Wake-on-LAN на Rust (Axum + SeaORM + PostgreSQL). Хранит список компьютеров и по запросу отправляет Magic Packet. Сделан для домашней лаборатории (Unraid, Docker с host networking).

### Возможности

- `GET /health` — публичная проверка живости.
- `GET /computers` — список компьютеров (нужен API-ключ).
- `POST /computers/{id}/wake` — отправить Magic Packet на broadcast-адрес компьютера, UDP-порт 9 (нужен API-ключ).
- API-ключ сравнивается за постоянное время (SHA-256 + `subtle`).
- Таблица `computers` создаётся автоматически при старте.

### Конфигурация

| Переменная | Обязательна | Описание |
|---|---|---|
| `API_KEY` | да | Токен в заголовке `X-Api-key`, не может быть пустым. |
| `DATABASE_URL` | да | Строка подключения к PostgreSQL. |
| `PORT` | нет | HTTP-порт, по умолчанию `8084` (`HOST_PORT` в `docker-compose.yml`). |

Без `API_KEY` или `DATABASE_URL` сервис завершается с понятной ошибкой. Пример — `.env.example`.

### Быстрый старт

Нужен Linux с Docker (host networking; в Docker Desktop для Mac/Windows UDP broadcast не доходит до локальной сети).

```bash
git clone https://github.com/Rip-07/wake-on-lan && cd wake-on-lan
cp .env.example .env                      # задайте API_KEY (openssl rand -hex 32)
docker compose --profile db up -d --build # сервис + PostgreSQL
```

Если PostgreSQL уже есть, укажите его в `DATABASE_URL` и запускайте без `--profile db`.

Нужен `network_mode: host`: UDP broadcast не выходит из bridge-сети Docker.

### Добавить компьютер

```bash
docker compose exec db psql -U wol -c \
  "INSERT INTO computers (name, mac_address, broadcast_address) VALUES ('Рабочий ПК', 'AA:BB:CC:DD:EE:FF', '192.0.2.255');"
```

### API

```bash
curl -H "X-Api-key: $API_KEY" http://127.0.0.1:8084/computers
curl -X POST -H "X-Api-key: $API_KEY" http://127.0.0.1:8084/computers/1/wake
```

### Разработка

```bash
cargo fmt --all --check
cargo clippy --all-targets --all-features --locked -- -D warnings
cargo test --locked
```

### Лицензия

MIT — см. [LICENSE](LICENSE).
