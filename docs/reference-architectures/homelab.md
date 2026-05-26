# Эталонная архитектура (Reference Architecture): Homelab

**Полностью приватно, на вашем собственном железе.** Ничто не покидает вашу LAN, кроме LLM-трафика к провайдеру (provider) — и опционально даже его нет, если вы запускаете локальные модели.

## Для кого это

- У вас есть homelab / NAS / выделенный сервер
- Приватность превыше всего — вы не хотите отправлять рецепты / PR / сообщения в стороннее облако
- Готовы пожертвовать удобством (динамический DNS, обновления) ради контроля

## Стоимость

- **Инфраструктура:** электричество + существующее железо
- **LLM:** $0 если всё через Ollama; иначе розничные API для выбранного подмножества
- **Внешние сервисы:** $0 (Tailscale Pro не требуется для 1–3 узлов)

## Архитектура (Architecture)

```
                    ┌──────────────────────────────────────┐
                    │            Homelab (LAN)             │
                    │                                      │
   phone / laptop → │  Tailscale    hermes.lan (Caddy)     │
   (Tailscale)      │     │              │                 │
                    │     └──────────────┤                 │
                    │                    ↓                 │
                    │         hermes.service (systemd)     │
                    │                ├── Ollama (GPU box)  │
                    │                ├── LightRAG          │
                    │                ├── Langfuse (self)   │
                    │                └── Dashboard :8765   │
                    │                                      │
                    └──────────────────────────────────────┘
                              │
                              ↓ (optional, for hard queries)
                          Anthropic / Google / OpenAI
```

## Состав

- **1× Linux-сервер** (16GB+ RAM, любой x86_64 или Apple Silicon VM) — запускает Hermes, LightRAG, Langfuse
- **1× GPU-сервер** (опционально; 16GB+ VRAM) — запускает Ollama. Может быть тем же сервером, если у вас один GPU.
- **Tailscale** (бесплатный тариф, до 3 пользователей / 100 устройств) — mesh VPN; без проброса портов
- **Домен** (опционально; `hermes.lan` отлично работает с Tailscale MagicDNS)

## Установка

### 1. Базовый сервер

```bash
# On the Linux box (as root, Debian 12 or Ubuntu 24.04)
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | bash
```

### 2. Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up --accept-routes
tailscale cert hermes.$(tailscale status --json | jq -r '.MagicDNSSuffix')
```

### 3. Ollama (опционально — локальные модели)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.1:70b-instruct-q4_K_M
ollama pull qwen2.5-coder:32b
```

### 4. Конфигурация

Начните с [`templates/config/production.yaml`](../../templates/config/production.yaml), затем:

```yaml
models:
  default: ollama/llama3.1:70b-instruct-q4_K_M
  providers:
    ollama:
      base_url: http://gpu-box.tailnet-xxx.ts.net:11434
    anthropic:
      api_key: "${ANTHROPIC_API_KEY}"        # fallback for hard queries

  routing:
    - when: task == "reasoning"
      use: anthropic/claude-sonnet
    - when: task == "coding" && complexity == "high"
      use: anthropic/claude-sonnet

gateways:
  cli:
    enabled: true
  telegram:
    enabled: true
    bots:
      admin:
        token: "${TELEGRAM_ADMIN_BOT_TOKEN}"
        allowed_user_ids:
          - ${TELEGRAM_OWNER_ID}

memory:
  backend: lightrag
  lightrag:
    working_dir: /var/lib/hermes/lightrag
    llm_model: ollama/qwen2.5-coder:32b         # local extraction
    embedding_model: openai/text-embedding-3-small  # or local (bge-m3)
```

### 5. Langfuse self-host (observability внутри LAN)

```bash
cp templates/compose/langfuse-stack.yml /opt/
cp templates/compose/.env.langfuse.example /opt/.env.langfuse
# edit /opt/.env.langfuse → generate secrets
docker compose -f /opt/langfuse-stack.yml --env-file /opt/.env.langfuse up -d
```

Укажите в Hermes `telemetry.langfuse.host` на `http://127.0.0.1:3000`.

### 6. Навыки (Skills)

```bash
for skill in /opt/hermes-optimization-guide/skills/*/*/; do
  ln -sfn "$skill" "/home/hermes/.hermes/skills/$(basename $skill)"
done
hermes /reload
```

## Честные компромиссы

- **Задержка.** Локальный 70B Q4 ≈ 20–40 токен/с на 3090. Флагманский Sonnet ≈ 60–90 токен/с. Большинство «рабочих» запросов вы не заметите; программирование/глубокие рассуждения — заметите.
- **Качество.** Текущие открытые/локальные модели (Qwen Coder, Llama, модели класса Kimi) *близки* по многим задачам, *отстают* в длинном контексте + нюансированных рассуждениях. Маршрутизация позволяет отдавать сложные задачи Sonnet'у.
- **Обслуживание.** Вы поддерживаете сервер. Включите unattended-upgrades (bootstrap-скрипт это делает) и запланируйте ежемесячные перезагрузки.
- **Доступность.** Tailscale надёжен, но означает «нет Tailscale = нет Hermes». Держите резервный административный бот на телефоне или запустите крошечное облачное реле.
- **Резервное копирование.** Настройте [`nightly-backup`](../../skills/ops/nightly-backup/SKILL.md) на запись зашифрованных архивов на второй физический диск — не на тот же RAID-массив.

## Что можно пропустить

- Cloudflare / публичный TLS — Tailscale справляется с этим
- UFW-правила для 80/443 — нет публичных портов
- Платный Langfuse — self-host бесплатен для любого разумного объёма одного пользователя

## Когда переходить на следующий уровень

Вы достигаете предела этой конфигурации когда:
- Вам нужно больше 1–2 человек, использующих её (разграничение прав с локальными моделями становится неудобным)
- Вам нужны вебхуки, доступные из интернета (Stripe, GitHub и т.д.)
- Ваш граф LightRAG превышает ~200K сущностей (он всё ещё будет работать, но слияния замедлятся)

Переходите на [Solo Developer](./solo-developer.md) (добавьте tiny VPS) или [Small Agency](./small-agency.md).
