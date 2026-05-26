# Эталонная архитектура (Reference Architecture): Road Warrior

**Телефон управляет, одноразовые облачные боксы делают тяжёлую работу.** Вдохновлено [Частью 21](../../part21-remote-sandboxes.md). У вас есть крошечный VPS за $5, работающий постоянно; он оркестрирует Modal / Daytona / Fly sandbox'ы, которые запускаются по требованию для реальной работы.

## Для кого это

- Путешествующие разработчики / номады
- Люди, которые программируют с телефона через Telegram
- Все, кто хочет энергетику «я могу починить прод из поезда»

## Стоимость

- **Постоянно работающий управляющий сервер:** $5/мес (Hetzner CX22)
- **Удалённые вычисления по требованию:** $0–50/мес (платите только когда реально запускаете задачи)
- **LLM:** $20–60/мес

## Архитектура (Architecture)

```
 Phone (Telegram) ──→ Driver VPS ($5/mo, always-on)
                            │
                            │   hermes.service
                            │   remote_sandbox: modal (default)
                            │
                            ▼
                     On-demand sandbox:
                       Modal (GPU-ish)
                       Daytona (full dev env)
                       Fly Machines (persistent)
                       E2B (Python sandbox)
                       SSH (your own beast)
```

Ваш телефон → Telegram → VPS за 5¢/мес → запускает Modal sandbox за $0.05/час → выполняет Claude Code, клонирует репозиторий, делает работу → синхронизирует файлы обратно при остановке → отправляет PR.

## Состав

- **Hetzner CX22** в качестве управляющего сервера ($5/мес)
- **Аккаунт Modal** (бесплатные $30/мес кредиты) ИЛИ **Daytona** ИЛИ **Fly Machines** — см. [Часть 21](../../part21-remote-sandboxes.md)
- **Telegram-бот** + ваш user ID
- **API-ключи:** Anthropic (для Claude Code внутри sandbox'а), опционально Google (для триажа Hermes на управляющем сервере)

## Установка

```bash
# On the driver VPS — as root
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | bash
```

Затем настройте:

```yaml
# /home/hermes/.hermes/config.yaml
version: 1

models:
  default: google/gemini-3.1-flash          # Cheap + fast for "plan the work" phase
  providers:
    google:
      api_key: "${GOOGLE_API_KEY}"
    anthropic:
      api_key: "${ANTHROPIC_API_KEY}"       # Used by sandboxed Claude Code

gateways:
  cli: { enabled: true }
  telegram:
    enabled: true
    bots:
      admin:
        token: "${TELEGRAM_ADMIN_BOT_TOKEN}"
        allowed_user_ids:
          - ${TELEGRAM_OWNER_ID}

# The money section
remote_sandbox:
  default_backend: modal          # Or daytona / fly / e2b / ssh
  backends:
    modal:
      token_id: "${MODAL_TOKEN_ID}"
      token_secret: "${MODAL_TOKEN_SECRET}"
      image: "python:3.12-slim"
      timeout_idle: 600           # 10m idle → auto-shutdown
    ssh:                          # your home beast, if any
      host: "beast.tailnet-xxx.ts.net"
      user: "hermes"
      identity_file: "~/.ssh/id_ed25519"

# Hermes loads skills from here; these let you orchestrate from Telegram
skills:
  allowlist:
    - pr-review
    - release-notes
    - cost-report
    - remote-run          # triggers a sandbox
```

## Рабочий процесс (Workflow)

```
you: "@bot fix the null-check in auth.ts"
bot:  [spinning up modal sandbox…]
bot:  cloned acme/app, branch devin-123
bot:  claude code: analyzing…
bot:  [file diff preview, 3 lines]
      Approve? /yes /no /changes
you:  /yes
bot:  [syncing files back, running tests]
bot:  tests green. Pushed PR #342 → https://…
bot:  sandbox torn down (ran 4m 12s, $0.014)
```

## Ключевые преимущества из Части 21 + PR #8018

- **Пакетная tar-pipe синхронизация** — холодный старт за 30s против 5 минут при 100× `scp`
- **SIGINT-безопасная обратная синхронизация** — потеряйте сигнал на полпути, sandbox всё равно сохранит данные при остановке
- **Синхронизация только по хешам** — возвращаются только изменённые файлы, а не всё дерево
- **Локальный `git push`** — управляющий VPS хранит ваши аутентифицированные git-учётные данные; sandbox их никогда не видит

## Настройка навыков (Skill setup)

```bash
# Symlink all the guide skills
for s in /opt/hermes-optimization-guide/skills/*/*/; do
  ln -sfn "$s" "/home/hermes/.hermes/skills/$(basename $s)"
done

# Write a tiny remote-run skill (paste into ~/.hermes/skills/remote-run/SKILL.md)
# that wraps `hermes sandbox run --repo acme/app -- claude -p "$@"`
hermes /reload
```

## Защитные ограничения (Safety rails)

- Sandbox = **карантинный профиль (quarantine profile)** (как если бы это был ненадёжный ввод) — Claude Code в sandbox'е не может получить доступ к MCP-серверам или секретам управляющего сервера
- У управляющего сервера есть GitHub PAT только на чтение (для триажа/поиска)
- **Записывающий (write)** PAT существует только внутри sandbox'а, живёт недолго, передаётся через stdin и никогда не сохраняется на диск

## Реальные затраты

Типичный месяц для активного пользователя:

| Статья | Стоимость |
|---|---:|---:|
| Управляющий сервер CX22 | $5 |
| Вычисления Modal (3ч/день × 30 дней × $0.05/ч) | $4.50 |
| Anthropic (Claude Code, с маршрутизацией) | $20–40 |
| Google Gemini Flash (триаж) | ~$0.50 |
| **Итого** | **~$30–50/мес** |

## Когда переходить на следующий уровень

- Вы запускаете 10+ часов sandbox'ов в день → мигрируйте на постоянную Fly Machine + масштабируйтесь
- Вам нужен GPU в sandbox'е → Modal A10G ~$1.10/ч, всё ещё дёшево для эпизодического использования
- Вам нужна *многопользовательская* работа → [Small Agency](./small-agency.md)
