# wake-on-lan

Small Wake-on-LAN HTTP service in Rust (Axum + SeaORM + PostgreSQL). It stores a list of computers and sends a Magic Packet to one of them on request. Built for a homelab (Unraid, Docker with host networking).

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

## Run

```bash
cp .env.example .env        # set API_KEY and DATABASE_URL
docker compose up -d --build
```

Host networking (`network_mode: host`) is required: UDP broadcast does not leave a bridged Docker network.

From source:

```bash
export API_KEY=... DATABASE_URL=postgres://...
cargo run --release
```

## Add a computer

```sql
INSERT INTO computers (name, mac_address, broadcast_address)
VALUES ('Workstation', 'AA:BB:CC:DD:EE:FF', '192.0.2.255');
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

### Запуск

```bash
cp .env.example .env        # задайте API_KEY и DATABASE_URL
docker compose up -d --build
```

Нужен `network_mode: host`: UDP broadcast не выходит из bridge-сети Docker.

### Добавить компьютер

```sql
INSERT INTO computers (name, mac_address, broadcast_address)
VALUES ('Рабочий ПК', 'AA:BB:CC:DD:EE:FF', '192.0.2.255');
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
