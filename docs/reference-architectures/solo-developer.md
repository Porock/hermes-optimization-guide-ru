# Эталонная архитектура (Reference Architecture): Solo Developer

**VPS + телефонный бот + маршрутизация стоимости.** Конфигурация на 80% случаев — дёшево, надёжно, доступно откуда угодно, достаточно для реальной работы одного человека.

## Для кого это

- Разработчик / мейкер, которому нужен Telegram/Discord драйвер для повседневной работы
- Нет команды поддержки; цель дизайна — «настроил и забыл»
- Готов тратить $5–50/мес, чтобы не обслуживать собственное железо

## Стоимость

- **Инфраструктура:** $5–7/мес (Hetzner CX22 или Fly machine)
- **LLM:** $20–60/мес для типичного личного использования с маршрутизацией стоимости
- **Домен + DNS:** $0–1/мес

**Итого: ~$25–70/мес.**

## Архитектура (Architecture)

```
  phone/laptop                Internet             hermes.yourdomain.com
       │                         │                         │
       └── Telegram/Discord ─────┼── Cloudflare/Caddy ────→│
                                 │                         │
                                 │                         ├── hermes.service
                                 │                         ├── hermes-dashboard.service
                                 │                         ├── Langfuse (self-host)
                                 │                         └── LightRAG
                                 │
                                 └── Anthropic / Google / Moonshot / Cerebras
```

## Состав

- **Hetzner CX22** (Debian 12 или Ubuntu 24.04) — $5/мес, 4GB RAM, 2 vCPU
- **Домен** ($12/год) — или используйте бесплатный поддомен от duckdns/nip.io
- **Токен Telegram-бота** (бесплатно; [@BotFather](https://t.me/BotFather))
- **API-ключи:** Anthropic (по умолчанию), Google (Gemini Flash для триажа), опционально Moonshot + Cerebras для программирования/классификации

## Установка

```bash
# As root on a fresh VPS
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | bash
```

Затем:

```bash
# As root
sudo -u hermes nano ~/.hermes/.env          # fill in keys
sudo cp /opt/hermes-optimization-guide/templates/config/cost-optimized.yaml \
     /home/hermes/.hermes/config.yaml       # or telegram-bot.yaml
sudo cp /etc/caddy/Caddyfile.hermes.reference /etc/caddy/Caddyfile
# edit /etc/caddy/Caddyfile — replace *.yourdomain.com
sudo systemctl reload caddy
sudo systemctl start hermes hermes-dashboard
```

## Почему `cost-optimized.yaml` — правильный выбор по умолчанию

См. [`templates/config/cost-optimized.yaml`](../../templates/config/cost-optimized.yaml). По умолчанию использует Gemini Flash (самая дешёвая умная модель), Cerebras Llama для классификации (почти бесплатно), и переключается на Sonnet только для сложных задач программирования. С кэшированием подсказок (prompt caching) + отключённым Fast Mode типичная стоимость составляет $0.05–0.30 за активный час.

Если вам нужно максимальное качество для конкретной задачи, просто скажите «используй sonnet» в чате — маршрутизатор учитывает явные переопределения пользователя.

## Навыки (Skills), которые вы будете запускать

Каждый из них устанавливается bootstrap-скриптом (симлинки в `~/.hermes/skills/`):

- Утро: `/cost-report window=24h` — расходы за вчера
- В фоновых потоках: `/telegram-triage` (автоответ)
- Еженедельно: `/weekly-dep-audit severity_floor=high`
- Еженощно: `/nightly-backup s3://my-backups/hermes/ 30` (или укажите `remote=local`, если вам всё равно)

## Пределы масштабирования

| Ограничение | Достигается при | Решение |
|---|---|---|
| RAM CX22 | ~5–10 конкурентных вызовов инструментов (tool calls) + LightRAG | Обновить до CX32 ($12/мес) |
| Бесплатный тариф Gemini Flash | 1500 запросов/день | Направить на Cerebras или добавить платную квоту |
| LightRAG на 2 vCPU | Индексация документов 10MB+ | Перенести индексацию в спотовую Modal sandbox |
| Бюджет на LLM | $50+/мес | Включить `prefer_cached: true` + порог сжатия 32K |

## Замечание по безопасности

Поскольку этот сервер доступен из интернета, **всегда** разворачивайте денлист (denylist) + require_approval из `cost-optimized.yaml`, и держите своего Telegram-бота **приватным** (ограничьте `allowed_user_ids` своим собственным ID). Любой «публичный» бот должен использовать отдельный токен и работать в **карантинном профиле (quarantine profile)** — см. [Часть 19](../../part19-security-playbook.md) и шаблон [`security-hardened.yaml`](../../templates/config/security-hardened.yaml).

## Когда переходить на следующий уровень

- Добавление коллег → [Small Agency](./small-agency.md)
- Переход на офлайн-режим → [Homelab](./homelab.md)
- Потребность в мощном облачном сервере по требованию → [Road Warrior](./road-warrior.md)
