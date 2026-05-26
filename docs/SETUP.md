# Подробная настройка

Если n8n ещё не установлен — сначала [DOCKER.md](DOCKER.md) (`docker compose up -d`).

## 1. Telegram-бот

1. Откройте [@BotFather](https://t.me/BotFather).
2. `/newbot` → имя и username.
3. Скопируйте **HTTP API token**.

В n8n: **Credentials → Telegram API** → вставьте token.  
Привяжите ко всем узлам типа `Telegram` и к **Telegram Trigger**.

## 2. OpenRouter

1. Зарегистрируйтесь на [openrouter.ai](https://openrouter.ai).
2. Создайте ключ: [openrouter.ai/keys](https://openrouter.ai/keys).
3. Пополните баланс (image-модели платные).

В n8n: **Credentials → Header Auth** (или Generic Credential Type → Header Auth):

| Поле | Значение |
|------|----------|
| Name | `Authorization` (или как требует ваша версия n8n) |
| Value | `Bearer sk-or-v1-...` |

Привяжите credential к:

- **Nano Banana 2 OpenRouter** (HTTP Request)
- **Редактор_изображений** (Tool Code)

Оба узла должны использовать **один и тот же** credential.

## 3. SerpAPI

1. [serpapi.com](https://serpapi.com) → API key.
2. В n8n: credential **SerpAPI** → привязать к узлу **SerpAPI** (tool агента).

## 4. Импорт workflow

1. **Workflows → Import from File**
2. Выберите [`../workflows/n8n-telegram-openrouter-agent.json`](../workflows/n8n-telegram-openrouter-agent.json)
3. Откройте workflow — проверьте узлы с предупреждением (красные) и назначьте credentials.

## 5. Webhook и активация

- **Docker:** в `.env` задайте `WEBHOOK_URL=https://ваш-публичный-домен/` (см. [DOCKER.md](DOCKER.md), ngrok или VPS).
- **Без Docker:** переменная `N8N_WEBHOOK_URL` / `WEBHOOK_URL` на хосте n8n — публичный HTTPS URL с `/` в конце.
- **Activate** workflow (переключатель Active).
- В Telegram напишите боту `/start`.

## 6. Проверка сценариев

| Тест | Ожидание |
|------|----------|
| `Привет` | Текстовый ответ агента |
| `нарисуй красный куб на белом фоне` | Фото в чат (~30–90 с) |
| Фото + подпись «сделай сепию» | Отредактированное фото |
| `Какая погода в Moscow?` | Ответ через wttr (латиница в городе) |

При ошибках откройте **Executions** в n8n и посмотрите узел, на котором остановилось выполнение.

## Переменные окружения (справочно)

См. [.env.example](../.env.example). Реальные ключи в n8n хранятся в **Credentials**, не в `.env`, если вы не настраиваете кастомную интеграцию.
