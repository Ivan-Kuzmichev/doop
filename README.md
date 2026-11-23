# doop-docker

Автосборка Docker-образа [Doop](https://doop.design) из релизов upstream-репозитория
[kgoedecke/doop](https://github.com/kgoedecke/doop).

Образ: **`ghcr.io/ivan-kuzmichev/doop`**

```sh
docker pull ghcr.io/ivan-kuzmichev/doop:latest
```

## Как работает

`.github/workflows/build-doop.yml` раз в 3 часа проверяет последний стабильный релиз `vX.Y.Z`
в upstream. Если образа с такой версией ещё нет в GHCR, он собирает его по Dockerfile из upstream и пушит:

- `ghcr.io/ivan-kuzmichev/doop:<версия>` (например `0.15.0`)
- `ghcr.io/ivan-kuzmichev/doop:<major.minor>` (например `0.15`)
- `ghcr.io/ivan-kuzmichev/doop:latest`

Вручную: **Actions → Build Doop image → Run workflow**. Можно указать конкретный тег (`v0.14.0`)
или включить `force`, чтобы пересобрать уже собранный. Ручная сборка конкретного тега не трогает `latest`.

## Настройки

- `PLATFORMS` в workflow: по умолчанию `linux/amd64`. Для ARM-сервера добавьте `linux/arm64`.
- Repository variable `VITE_POSTHOG_KEY` (необязательно): ключ аналитики PostHog, вшивается в клиент.

## Запуск на сервере

Минимальный `docker-compose.yml` (Doop + Postgres):

```yaml
services:
  doop:
    image: ghcr.io/ivan-kuzmichev/doop:latest
    ports:
      - '4400:4400'
    environment:
      DATABASE_URL: postgres://doop:doop@db:5432/doop
      BETTER_AUTH_SECRET: ${BETTER_AUTH_SECRET:?set BETTER_AUTH_SECRET}
      BETTER_AUTH_URL: ${BETTER_AUTH_URL:-http://localhost:4400}
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: doop
      POSTGRES_PASSWORD: doop
      POSTGRES_DB: doop
    volumes:
      - doop-pg:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  doop-pg:
```

```sh
BETTER_AUTH_SECRET=$(openssl rand -hex 32) docker compose up -d
# → http://localhost:4400
```

Полная версия с healthcheck и дополнительными переменными (`SIGNUP_EMAIL_DOMAINS`,
`ANTHROPIC_API_KEY`) лежит в [`docker-compose.yml`](docker-compose.yml).

`BETTER_AUTH_SECRET` должен быть постоянным: сгенерируйте его один раз и сохраните в `.env`
рядом с compose-файлом, иначе после перезапуска все сессии слетят.

Если пакет в GHCR приватный, сначала выполните `docker login ghcr.io`
(с Personal Access Token с правом `read:packages`) или сделайте пакет публичным в его настройках.
