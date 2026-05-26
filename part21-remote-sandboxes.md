# Часть 21: Удалённые песочницы (Remote Sandboxes) и массовая синхронизация файлов — SSH, Modal, Daytona, Vercel

*Запускать Hermes на VPS за $5 отлично для чата. Запускать там тяжёлую работу по кодингу — нет. Эта часть настраивает паттерн «телефон управляет, мощный удалённый сервер делает работу»: Hermes живёт на вашем маленьком VPS, делегирует выполнение в одноразовую песочницу на SSH/Modal/Daytona/Vercel, синхронизирует файлы в обе стороны и уничтожает её при простое.*

---

## Паттерн

```
Your phone (Telegram)
        │
        ▼
Hermes on $5 VPS  ─────────────►  Remote sandbox ($0 when idle)
- Memory                            - Whole workspace in /home/runner/
- Skills                            - Coding agents (Claude/Codex/etc)
- Conversation state                - Build tools, Docker, GPU
        ▲                                │
        │                                │
        └─── bulk file sync on teardown ─┘
```

Hermes загружает ваше рабочее пространство при старте задачи, делегирует работу, а затем скачивает только diff обратно при остановке. Песочница умирает, Hermes сохраняет состояние — и вашему VPS за $5 никогда не понадобились 32 ГБ ОЗУ, в которых работала песочница.

---

## Выбор бэкенда

| Бэкенд | Тарификация | Стоимость простоя | Лучше всего для |
|---------|---------|-----------|----------|
| **SSH** | Ваша инфраструктура | Сколько стоит ваш хостинг | Домашняя лаборатория / постоянно включённая dev-машина |
| **Modal** | Посекундная оплата вычислений | $0 (гибернация) | Периодические задачи по кодингу, работа с GPU |
| **Daytona** | Посекундная оплата рабочего пространства | $0 (гибернация) | Долгоживущие dev-рабочие пространства |
| **Vercel Sandbox** | За запуск / платформенная тарификация | $0 при отсутствии использования | Сборки веб-приложений и изолированные задачи `execute_code` |
| **Fly Machines** | Посекундная | $0 (остановка) | Региональные песочницы рядом с пользователями |
| **E2B** | Посекундная | $0 | Быстрые одноразовые Python-песочницы |
| **Local Docker** | Ваше оборудование | Н/Д | Тестирование / разработка |

Hermes имеет встроенную поддержку SSH, Modal, Daytona и Vercel Sandbox. Fly Machines и E2B работают через тонкие плагины (plugins).

---

## SSH-бэкенд (Домашняя лаборатория / Постоянно включённая dev-машина)

### Требования

- SSH-доступ к удалённому хосту с аутентификацией по ключу (без запроса пароля)
- На удалённом хосте установлены `python3`, `rsync`, `tar`, `git`
- Ваш SSH-конфиг использует `ControlMaster` + `ControlPath` для переиспользования соединения (показано ниже)

### Конфигурация

```yaml
# ~/.hermes/config.yaml
sandboxes:
  dev-box:
    backend: ssh
    host: dev.local
    user: hermes
    identity_file: ~/.ssh/hermes_ed25519
    workdir: /home/hermes/sandboxes
    control_master: auto              # Reuses connection for bulk sync
    control_persist: 600
    sync:
      push: ~/.hermes                 # Uploaded at sandbox create
      pull_on_teardown: true
      pull_paths:
        - .hermes
        - projects                    # Grabs any code changes made in-sandbox
      ignore:
        - .git
        - node_modules
        - __pycache__
        - "*.log"
```

### Использование

```
/sandbox start dev-box
/claude-code refactor src/auth/ to use JWT rotation
/sandbox stop dev-box                # Syncs changes back, then stops
```

Что происходит под капотом при остановке:

1. Hermes запускает `tar cf - -C ~/.hermes .` на удалённом сервере
2. Передаёт его через SSH ControlMaster на локальную машину
3. Распаковывает в промежуточную директорию
4. Сравнивает с SHA-256-хешами того, что было изначально отправлено
5. Применяет только изменённые файлы обратно в `~/.hermes`, с сериализацией через `fcntl.flock` если другая песочница работает одновременно
6. Устойчиво к SIGINT — нажатие Ctrl-C во время синхронизации чисто откатывает изменения

Это укрепление защиты сделало удалённые песочницы достаточно безопасными для реальной работы с кодом. До синхронизации на основе diff у вас был выбор: либо rsync'ить всё каждый раз (медленно), либо терять изменения, сделанные на удалённом сервере, при остановке.

---

## Modal-бэкенд (Периодические задачи / Serverless)

Modal переводит песочницы в режим гибернации до нуля между запусками и поднимает их за ~2 секунды. Идеально для периодического использования агентами (agents) для кодинга.

```bash
pip install modal
modal token new
```

```yaml
sandboxes:
  modal-big:
    backend: modal
    image:
      from: python:3.12
      apt_install: [git, ripgrep, build-essential]
      pip_install: [claude-code-cli, aider-chat]
    cpu: 4
    memory: 16384
    gpu: null                        # Set to "T4" / "A10G" / "H100" if you need one
    timeout: 3600
    sync:
      push: ~/.hermes
      pull_on_teardown: true
      pull_paths: [.hermes, projects]
```

Синхронизация использует Modal `exec tar cf -` → `proc.stdout.read()` → локальный файловый паттерн — та же логика diff/apply, что и у SSH.

Совет по стоимости: установите `timeout: 300` и короткий `idle_shutdown:` для чат-ориентированных песочниц; Modal тарифицирует посекундно за фактическое время работы.

### GPU-песочницы для голосовых / графических задач

Если вы отключили [шлюз инструментов (Tool Gateway)](./part13-tool-gateway.md) и запускаете собственный пайплайн генерации изображений или голоса, GPU-песочница обойдётся дешевле, чем постоянная GPU VPS:

```yaml
sandboxes:
  gpu-a10g:
    backend: modal
    image:
      from: nvcr.io/nvidia/pytorch:24.10-py3
      pip_install: [diffusers, transformers]
    gpu: "A10G"
    timeout: 600
    commands:
      - /generate_image    # Route image gen to this sandbox
      - /speech_synth
```

Hermes маршрутизирует вызовы инструментов прозрачно — пользователь даже не знает, что песочница была развёрнута.

---

## Daytona-бэкенд (Долгоживущие рабочие пространства)

Daytona — это вариант «как GitHub Codespaces для вашего собственного кода». Используйте его с Hermes, когда хотите, чтобы рабочее пространство сохранялось между сессиями (sessions):

```yaml
sandboxes:
  workspace:
    backend: daytona
    workspace_id: hermes-dev
    auto_create: true                # Create if it doesn't exist
    image: daytonaio/workspace-project:latest
    hibernate_after: 900
    sync:
      push: ~/.hermes
      pull_on_teardown: false        # Work persists, no need to sync every time
      pull_on_command: "/sync-home"  # Manual sync when you want it
```

Объедините с [провайдером (provider) Gemini OAuth](./part9-custom-models.md#gemini-oauth--free-tier-friendly) для чтения длинного контекста внутри песочницы на бесплатном тарифе.

---

## Vercel Sandbox (Веб-сборки / Изолированное выполнение кода)

Vercel Sandbox теперь является нативным бэкендом для `execute_code` и терминальных запусков. Используйте его, когда задача связана с веб-приложением: установить зависимости, запустить сборку, просмотреть сгенерированный вывод и выбросить окружение.

```yaml
sandboxes:
  vercel-web:
    backend: vercel
    project: my-webapp
    timeout: 1800
    sync:
      push: ~/projects/my-webapp
      pull_on_teardown: true
      pull_paths:
        - .
      ignore:
        - node_modules
        - .next
        - dist
```

Это не замена Daytona, если вам нужно постоянное рабочее пространство для разработки. Воспринимайте его как чистое окружение для сборок, тестов и коротких изолированных скриптов.

---

## Fly Machines (Региональные / Низкая задержка)

Для пользователей в определённых регионах Fly Machines обеспечивают задержку менее 100 мс из ближайшей точки присутствия (PoP):

```yaml
sandboxes:
  fly-sin:
    backend: fly_machines             # Plugin, not core
    app: hermes-sandbox
    region: sin                       # Singapore
    size: performance-2x
    auto_stop: true
    stopped_shutdown_at: 120
```

Полезно, когда нужно, чтобы песочница находилась физически рядом с вашими iOS/Telegram-пользователями для меньшего времени оборота.

---

## E2B (Одноразовые Python-песочницы)

E2B предоставляет чистую Linux-песочницу примерно за 500 мс. Лучше всего подходит для анализа данных / выполнения неизвестного кода:

```yaml
sandboxes:
  e2b-scratch:
    backend: e2b
    template: python                  # E2B template
    metadata:
      purpose: data-analysis
    timeout: 300
```

Hermes маршрутизирует любой вызов инструмента, помеченный `/sandbox e2b`, в этот шаблон. Остановка происходит автоматически.

---

## Кросс-песочничные паттерны (Cross-Sandbox Patterns)

### Паттерн A: Основная-реплика Dev Box + Эфемерные песочницы

- **Основная:** SSH dev-машина с вашим долгоживущим рабочим пространством
- **Реплика:** Modal-песочница, запускаемая при каждой делегации

```
/sandbox start dev-box
/delegate (runs in modal-big, reads from dev-box via git)
/sandbox stop dev-box
```

Отлично работает, когда каждая делегация агенту для кодинга запускает git-ветку с функциями. Песочницы не имеют состояния; dev-box является источником истины.

### Паттерн B: Проектные Daytona-рабочие пространства

```
/project open myapp       → daytona workspace "myapp"
/project open sideproject → daytona workspace "sideproject"
```

Каждый проект имеет собственное рабочее пространство со своими зависимостями, окружением и git-состоянием. Hermes запоминает, какое активно для каждой Telegram-темы.

### Паттерн C: Песочничные MCP-серверы (Sandboxed MCP Servers)

Направляйте ненадёжные MCP-серверы (см. [Часть 19](./part19-security-playbook.md#mcp-server-trust-model)) в песочницу:

```yaml
mcp_servers:
  random-scraper:
    trust: untrusted
    run_in_sandbox: e2b-scratch       # Isolate execution
```

Песочница перехватывает любое вредоносное поведение — даже если скрейпер скомпрометирован, он не сможет добраться до вашего хоста.

---

## Наблюдаемость: `hermes sandbox status`

```
$ hermes sandbox status
NAME         BACKEND   STATE      AGE      CPU   MEM      COST
dev-box      ssh       connected  3h 12m   0.4   2.1 GB   $0 (your infra)
modal-big    modal     running    0m 42s   3.8   14.2 GB  $0.09
workspace    daytona   hibernated 0m 0s    -     -        $0
```

[Веб-панель](./part12-web-dashboard.md) имеет панель песочниц с той же информацией плюс: потоковые логи, общую стоимость по каждой песочнице за месяц, историю синхронизации и кнопку «синхронизировать и остановить» в один клик.

---

## Устранение неполадок

| Симптом | Исправление |
|---------|-----|
| Тайм-аут остановки песочницы во время синхронизации | Увеличьте `sync.timeout: 600` — большие рабочие пространства по медленному SSH |
| Конфликт синхронизации: файл на хосте также изменился | По умолчанию побеждает последняя запись; установите `sync.conflict: prompt` для интерактивного разрешения |
| Сокет SSH ControlMaster занят | Запущен другой процесс Hermes на машине; `hermes sandbox ps` чтобы найти его |
| Холодный старт Modal-песочницы постоянно завершается по тайм-ауту | Предварительно прогрейте с помощью `hermes sandbox warm modal-big` перед началом интерактивной работы |
| Гибернация Daytona → возобновление повреждает git-состояние | Добавьте `.git` в `pull_paths`, чтобы Hermes хранил каноническую копию |
| Синхронизация файлов загружает .venv каждый раз | Добавьте его в `ignore:` — по умолчанию пропущен в некоторых шаблонах |

Включите `HERMES_SANDBOX_LOG=debug`, чтобы получить полные трассировки команд tar/ssh.

---

## Что дальше

- [Часть 18: Агенты для кодинга (Coding Agents)](./part18-coding-agents.md) — делегируйте Claude Code / Codex / Gemini CLI *в* эти песочницы
- [Часть 19: Практическое руководство по безопасности (Security Playbook)](./part19-security-playbook.md) — изолируйте ненадёжные MCP в песочницах
- [Часть 20: Наблюдаемость и стоимость (Observability & Cost)](./part20-observability.md) — отслеживайте затраты на песочницы вместе с расходами на LLM
- [Часть 1: Установка (Setup)](./README.md#part-1-setup-stop-fumbling-with-installation) — базовая установка VPS, которую они расширяют
