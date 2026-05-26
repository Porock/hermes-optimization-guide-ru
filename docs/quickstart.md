# Быстрый старт (quickstart) за 5 минут

С нуля до работающего Telegram-бота.

## Предварительные требования

- Машина с Linux, macOS или WSL (всё, где есть bash)
- Аккаунт Telegram
- API-ключ Anthropic для модели по умолчанию
- API-ключ Google — [aistudio.google.com](https://aistudio.google.com/apikey) для классификации Gemini Flash и LLM LightRAG в шаблоне Telegram
- API-ключ OpenAI — [platform.openai.com/api-keys](https://platform.openai.com/api-keys) для эмбеддингов LightRAG в шаблоне Telegram

## Шаг 1 — Установка Hermes

```bash
curl -sSL https://install.hermes.nous.ai | bash
hermes --version          # проверка работоспособности
```

## Шаг 2 — Создание Telegram-бота

1. Напишите [@BotFather](https://t.me/BotFather) → `/newbot` → следуйте инструкциям
2. Скопируйте токен бота
3. Напишите вашему новому боту что-нибудь (любое сообщение), чтобы он вас увидел
4. Узнайте ваш Telegram ID — напишите [@userinfobot](https://t.me/userinfobot)

## Шаг 3 — Установите конфиг

```bash
# Клонируйте гайд
git clone https://github.com/OnlyTerp/hermes-optimization-guide ~/hermes-guide

# Скопируйте шаблон Telegram-бота
mkdir -p ~/.hermes
cp ~/hermes-guide/templates/config/telegram-bot.yaml ~/.hermes/config.yaml
```

## Шаг 4 — Заполните секреты

Создайте `~/.hermes/.env`:

```bash
cat > ~/.hermes/.env <<'EOF'
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...                 # требуется telegram-bot.yaml для эмбеддингов LightRAG
GOOGLE_API_KEY=AIza...                # требуется telegram-bot.yaml для классификации Gemini Flash + LLM LightRAG
TELEGRAM_ADMIN_BOT_TOKEN=1234567890:ABC...
TELEGRAM_OWNER_ID=1234567            # ваш числовой ID от @userinfobot
EOF

chmod 600 ~/.hermes/.env
```

## Шаг 5 — Запустите

```bash
hermes run &
```

Напишите вашему боту. Он должен ответить через несколько секунд.

## Шаг 6 — Установите нужные вам навыки (skills)

```bash
for skill in ~/hermes-guide/skills/*/*/SKILL.md; do
  name=$(basename $(dirname "$skill"))
  ln -sfn "$(dirname "$skill")" "$HOME/.hermes/skills/$name"
done
hermes /reload
```

Теперь попробуйте:

- `/audit-mcp` — серверов пока нет, так что вы получите «нечего проверять» (ожидаемо)
- `/cost-report` — показывает использование токенов этой сессии (session)
- Задайте любой вопрос в свободной форме — чат работает сразу

## Шаг 7 — Прокачка

- **Больше платформ:** [Часть 4 (Telegram deep-dive)](../part4-telegram-setup.md), [Часть 15 (iMessage/WeChat/Android)](../part15-new-platforms.md)
- **Новейшие возможности:** [Часть 22 (Curator, TUI, плагины (plugins))](../part22-latest-power-moves.md), [Часть 23 (Kanban, `/goal`, Checkpoints v2)](../part23-tenacity-stack.md)
- **Память (Memory), которая рассуждает:** [Часть 3 (LightRAG)](../part3-lightrag-setup.md)
- **Инструменты (Tools):** [Часть 17 (MCP-серверы)](../part17-mcp-servers.md)
- **Драйвер кодинг-агентов (agent):** [Часть 18 (Claude Code, Codex, Gemini CLI)](../part18-coding-agents.md)
- **Production-закалка:** [Часть 19 (Security)](../part19-security-playbook.md) + [Часть 20 (Observability)](../part20-observability.md)
- **Установка на VPS одной командой:** [`scripts/vps-bootstrap.sh`](../scripts/vps-bootstrap.sh)

## Частые проблемы первого часа

| Симптом | Решение |
|---|---|
| Бот не отвечает | `journalctl --user -u hermes` — в 99% случаев отсутствует переменная окружения |
| 401 от Anthropic | Проверьте, что в `ANTHROPIC_API_KEY` нет завершающего перевода строки: `cat -A ~/.hermes/.env` |
| «skill not found: /cost-report» | Выполните `hermes /reload` после создания симлинков навыков |
| Медленные ответы | Вы на бесплатном тарифе Anthropic — действуют лимиты. Обновите тариф или перенаправьте на Gemini Flash через шаблон `cost-optimized` |
