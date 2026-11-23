# doop-docker

Автосборка Docker-образа [Doop](https://doop.design) из релизов upstream-репозитория
[kgoedecke/doop](https://github.com/kgoedecke/doop).

## Как работает

`.github/workflows/build-doop.yml` раз в 3 часа проверяет последний стабильный релиз `vX.Y.Z`
в upstream. Если образа с такой версией ещё нет в GHCR, он собирает его по Dockerfile из upstream и пушит:

- `ghcr.io/<owner>/doop:<версия>` (например `0.15.0`)
- `ghcr.io/<owner>/doop:<major.minor>` (например `0.15`)
- `ghcr.io/<owner>/doop:latest`

Вручную: **Actions → Build Doop image → Run workflow**. Можно указать конкретный тег (`v0.14.0`)
или включить `force`, чтобы пересобрать уже собранный.

## Настройки

- `PLATFORMS` в workflow: по умолчанию `linux/amd64`. Для ARM-сервера добавьте `linux/arm64`.
- Repository variable `VITE_POSTHOG_KEY` (необязательно): ключ аналитики PostHog, вшивается в клиент.

## Запуск на сервере

```sh
# в docker-compose.yml замените OWNER на свой GitHub-ник
BETTER_AUTH_SECRET=$(openssl rand -hex 32) docker compose up -d
# → http://localhost:4400
```

Если пакет в GHCR приватный, сначала выполните `docker login ghcr.io`
(с Personal Access Token с правом `read:packages`) или сделайте пакет публичным в его настройках.
