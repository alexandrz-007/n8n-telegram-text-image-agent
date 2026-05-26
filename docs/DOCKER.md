# Запуск n8n через Docker

Docker **поднимает только n8n**. Workflow, credentials и публичный URL для Telegram настраиваются отдельно (5–15 минут).

## Требования

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows/macOS) или Docker Engine (Linux)
- [Docker Compose](https://docs.docker.com/compose/) v2 (`docker compose version`)

## Быстрый старт

```powershell
cd путь\к\репозиторию
copy .env.example .env
docker compose up -d
```

Откройте в браузере: **http://localhost:5678**

Первый запуск: создайте учётную запись владельца n8n (локально).

Остановка:

```powershell
docker compose down
```

Данные n8n (workflows, credentials) сохраняются в Docker-volume `n8n_data`. Чтобы удалить и их:

```powershell
docker compose down -v
```

## Импорт workflow

1. В n8n: **Workflows → Import from File**
2. Файл в репозитории: [`workflows/n8n-telegram-openrouter-agent.json`](../workflows/n8n-telegram-openrouter-agent.json)

В контейнере тот же файл смонтирован read-only: `/home/node/import/n8n-telegram-openrouter-agent.json` (удобно, если импортируете из shell внутри контейнера).

Дальше — [SETUP.md](SETUP.md): Telegram, OpenRouter, SerpAPI, Activate workflow.

## Telegram и WEBHOOK_URL

Telegram шлёт события на **внешний HTTPS URL**. `http://localhost:5678` с вашего ПК **недоступен** серверам Telegram.

Варианты:

### A. VPS или сервер с доменом (продакшен)

На сервере с Docker задайте в `.env`:

```env
N8N_HOST=your-domain.com
N8N_PROTOCOL=https
N8N_EDITOR_BASE_URL=https://your-domain.com
WEBHOOK_URL=https://your-domain.com/
N8N_SECURE_COOKIE=true
```

Перед n8n — reverse proxy (Caddy, Nginx, Traefik) с TLS.

### B. Локальная разработка — туннель

1. Запустите `docker compose up -d`
2. Поднимите туннель, например [ngrok](https://ngrok.com):

   ```powershell
   ngrok http 5678
   ```

3. Скопируйте HTTPS URL (например `https://abc123.ngrok-free.app`)
4. В `.env`:

   ```env
   WEBHOOK_URL=https://abc123.ngrok-free.app/
   N8N_EDITOR_BASE_URL=https://abc123.ngrok-free.app
   N8N_SECURE_COOKIE=true
   ```

5. Перезапустите n8n:

   ```powershell
   docker compose up -d
   ```

6. **Activate** workflow в n8n и проверьте бота в Telegram.

При каждом новом URL ngrok обновляйте `WEBHOOK_URL` и перезапускайте контейнер.

## Переменные (.env)

| Переменная | Назначение |
|------------|------------|
| `N8N_PORT` | Порт на хосте (по умолчанию 5678) |
| `WEBHOOK_URL` | Базовый URL для webhooks (критично для Telegram Trigger) |
| `N8N_EDITOR_BASE_URL` | URL редактора n8n |
| `N8N_BASIC_AUTH_*` | Опциональная защита UI паролем |
| `TZ` | Часовой пояс |

Ключи `TELEGRAM_BOT_TOKEN`, `OPENROUTER_API_KEY`, `SERPAPI_API_KEY` в `.env.example` — **напоминание**; в Docker они **не подставляются** автоматически, их вводят в **Credentials** в UI n8n.

## Полезные команды

```powershell
docker compose logs -f n8n
docker compose ps
docker compose pull
docker compose up -d
```

## Устранение неполадок

| Проблема | Решение |
|----------|---------|
| Порт 5678 занят | В `.env` задайте `N8N_PORT=5679` и откройте `http://localhost:5679` |
| Бот молчит | Проверьте `WEBHOOK_URL`, HTTPS, Active workflow, credential Telegram |
| Healthcheck failing | Подождите 1–2 мин после старта; `docker compose logs n8n` |
| Windows: volume errors | Запустите Docker Desktop, включите WSL2 backend |

## Версия n8n

Образ: `docker.n8n.io/n8nio/n8n:latest`. Workflow рассчитан на **n8n 2.21.7+**. Для фиксации версии замените тег в [`docker-compose.yml`](../docker-compose.yml), например `docker.n8n.io/n8nio/n8n:1.82.1` (сверьте с [релизами n8n](https://docs.n8n.io/release-notes/)).
