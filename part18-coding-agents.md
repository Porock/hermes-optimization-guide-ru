# Часть 18: Делегирование агентам кодинга — Claude Code, Codex, Gemini CLI, OpenCode

*Коронный приём Hermes для разработчиков — не написание кода самим собой, а **оркестрация** специализированных агентов кодинга (coding agents) из Telegram-чата или Kanban-доски. Управляйте Claude Code, Codex, Gemini CLI, OpenCode и дешёвыми каналами Kimi/GLM со своего телефона, пока Hermes держит состояние, память (memory), утверждения (approvals) и шлюзы ревью (review gates).*

---

## Зачем делегировать, а не делать самому

Hermes отлично справляется с рассуждением, памятью, диалогом и рабочими процессами. Но он *не* лучший в долгой многфайловой генерации кода. Агенты, специализирующиеся на кодинге:

| Агент (agent) | Сильные стороны | Модель аутентификации |
|-------|-----------|------------|
| **Claude Code** | Лучшая безотказная работа с PR, крупные рефакторинги, тесты, ревью; работа в паре с Sonnet 5/Opus 4.7 | Pro/Max OAuth или `ANTHROPIC_API_KEY` |
| **Codex** (OpenAI) | Быстрый цикл в песочнице, охота на баги, правки малого/среднего размера; силён с GPT-5.5/Codex | OAuth через `openai` CLI или `OPENAI_API_KEY` |
| **Gemini CLI** | Контекст на 1M токенов и мультимодальное сканирование репозиториев/документов; самый мощный канал «сначала прочитай всё» | OAuth через `gemini auth`; собственный Gemini OAuth Hermes покрывает обычное использование провайдера (provider) моделей |
| **OpenCode** (anomalyco) | Открытый исходный код, маршрутизация к Kimi K2.6 / GLM / MiMo по низкой цене | Принесите свой ключ любого провайдера |
| **Aider** | Хирургические git-ориентированные правки, минимальный расход токенов | Принесите свой ключ любого провайдера |

Hermes хранит состояние, память, диалог, утверждения, жизненный цикл Kanban и интеграцию с платформами; каждый специалист делает то, что умеет лучше всего. Вы получаете одну панель управления, множество агентов.

---

## Предварительные требования

```bash
# Claude Code
npm install -g @anthropic-ai/claude-code
claude auth login                 # Или установите ANTHROPIC_API_KEY

# Codex
npm install -g @openai/codex-cli
codex auth login

# Gemini CLI
npm install -g @google/gemini-cli
gemini auth                       # Нужно только при делегировании самому Gemini CLI

# OpenCode (предпочтительна Go-версия для Hermes)
curl -fsSL https://opencode.ai/install.sh | bash
opencode auth                     # BYOK

# Aider
pipx install aider-chat
```

Проверка изнутри Hermes:

```
/skill claude-code
/skill codex
/skill gemini-cli
```

Каждый навык (skill) запускает `--version` и `auth status`, чтобы убедиться, что агент доступен.

---

## Режим 1: Print Mode (предпочтителен для большинства задач)

Print mode — неинтерактивный режим: однократный запуск, возврат результата, завершение. Без PTY, без окон подтверждения, чистый захват stdout. Идеален для 80% задач типа «вот изменение, вернись, когда будет готово».

### Через навык (рекомендуется)

Hermes поставляется с навыком `claude-code`, который управляет настройкой окружения, флагами разрешённых инструментов и восстановлением после ошибок:

```
/claude-code refactor src/auth/ to use the new JWT rotation helper
```

Это выполняет:

```bash
claude -p "refactor src/auth/ to use the new JWT rotation helper" \
       --allowedTools "Read,Edit,Bash" \
       --max-turns 20 \
       --output-format json
```

Захватывает JSON, парсит diff файлов, публикует сводку обратно в ваш Telegram/Discord/Slack-тред со ссылкой на git diff.

### Параллельное делегирование

Нужно сделать три дела? Запустите все три одновременно:

```
In parallel:
1. /claude-code write unit tests for src/payments/
2. /codex optimize the hot path in worker.ts
3. /gemini-cli audit dependencies in package.json for security
```

Hermes запускает их в трёх независимых слотах сабагентов (subagent), транслирует прогресс и агрегирует результаты.

### Маршрутизация по стоимости в зависимости от типа задачи

У каждого специалиста есть своя ниша. Позвольте Hermes маршрутизировать:

| Задача | Специалист | Почему |
|------|-----------------------|-----|
| Крупный рефакторинг в 10+ файлах | Claude Code + Sonnet 5/Opus 4.7 | Лучший в длительных многфайловых правках |
| Воспроизведение бага + исправление в одном файле | Codex + GPT-5.5/Codex | Быстрый цикл в песочнице |
| «Объясни эту кодовую базу» | Gemini CLI + Gemini 3.1 Pro | Контекст 1M токенов «съедает» любой репозиторий целиком |
| Массовые хирургические правки с детерминированными diffs | Aider | Минимальный расход токенов, нативная работа с git |
| Всё, что нужно сделать бюджетно | OpenCode + Kimi K2.6 / GLM | Намного дешевле фронтирных моделей для рутинных правок |

Разумный `~/.hermes/config.yaml`:

```yaml
delegation:
  default: claude-code
  routing:
    - match: { type: refactor, files_changed_gte: 5 }
      agent: claude-code
    - match: { type: bugfix, single_file: true }
      agent: codex
    - match: { type: explore, repo_tokens_gte: 200000 }
      agent: gemini-cli
    - match: { type: dependency_audit }
      agent: gemini-cli
    - match: { budget: low }
      agent: opencode
      model: moonshot/kimi-k2.6
```

## Режим 1B: Kanban Worker Lanes (предпочтительно для долгой работы)

Для работы, которая должна переживать перезапуски, человеческое ревью, повторные попытки или множественные передачи, поместите агента кодинга за [Kanban-поток из Части 23](./part23-tenacity-stack.md#2-add-worker-lanes-instead-of-giant-prompt-swarms):

```text
/kanban create "Fix flaky checkout tests and open a PR" \
  --assignee codex-worker \
  --workspace worktree
```

Хорошие значения по умолчанию:

- `codex-worker` для небольших изолированных исправлений; успешный выход блокируется для ревью Hermes/человеком вместо автозавершения.
- `claude-code` для многфайловых рефакторингов; требует тесты и ревью перед отметкой о завершении.
- `gemini-cli` для карточек аудита масштаба репозитория, которые должны создавать комментарии/спецификации, а не коммиты.
- `reviewer` как отдельный канал, чтобы состояния «агент написал код» и «работа завершена» оставались разными.

Используйте print mode для быстрых одноразовых ответов. Используйте Kanban-каналы для всего, что было бы обидно потерять на полпути.

---

## Режим 2: Привязанные к тредам интерактивные сессии (шаблон OpenClaw)

То, что вам на самом деле нужно на телефоне: тема Telegram под названием «Claude Code», где каждое сообщение попадает в постоянную сессию (session) Claude Code. Без повторного объяснения контекста. Без перезапусков. Просто общайтесь с агентом кодинга напрямую, а Hermes занимается транспортом, памятью и преобразованием голоса в текст.

Этот шаблон полезен для парного программирования из чата. Для безотказной работы предпочитайте Kanban-каналы воркеров (worker), чтобы состояние задачи и шлюзы ревью переживали перезапуски. Интерактивный рабочий процесс:

```bash
# В Telegram создайте тему, затем из CLI или панели управления:
hermes bind-thread <thread-id> --runtime claude-code --cwd ~/projects/myapp
```

С этого момента:
- Каждое сообщение в теме уходит в постоянную сессию Claude Code
- Правки файлов происходят в `~/projects/myapp` на хосте Hermes
- Оркестратор-сабагенты могут порождать собственных воркеров, если `max_spawn_depth` позволяет
- Конкурирующие воркеры координируют состояние файлов вместо слепой перезаписи собратьев
- `/unbind` в теме открепляет сессию и возвращает обычный чат с Hermes
- `/runtime gemini-cli` меняет среду выполнения без потери треда

Та же привязка работает для Codex, Gemini CLI, OpenCode и любого ACP-совместимого агента кодинга.

**Бонус удалённого выполнения:** сочетание с [функцией удалённых песочниц](./part21-remote-sandboxes.md) — агент кодинга работает на хосте Modal/Daytona/SSH, ваш телефон управляет, а всю работу делает мощный удалённый сервер.

---

## ACP: Протокол, который делает это возможным

Agent Client Protocol (ACP) — это для агентов кодинга то же, что MCP для инструментов (tools): стандартный транспорт для делегирования одного агента другому. Hermes поддерживает ACP и как клиент, и как сервер:

- **Как ACP-клиент:** Hermes вызывает Claude Code / Codex / Gemini в качестве сабагентов через их ACP-энпоинты.
- **Как ACP-сервер:** вы можете управлять Hermes из другого ACP-совместимого агента (Cursor, Zed или другой экземпляр Hermes).

```yaml
# ~/.hermes/config.yaml
acp:
  enabled: true
  server:
    listen: 127.0.0.1:41212          # Принимает входящий ACP от редакторов
  clients:
    claude-code:
      command: claude
      args: ["--acp"]
    codex:
      command: codex
      args: ["--acp"]
    gemini-cli:
      command: gemini
      args: ["--acp"]
```

Инструмент (tool) `/delegate_task` затем выбирает ACP-клиента на основе правил `delegation.routing` и транслирует прогресс через единый WebSocket.

---

## Git-гигиена, когда агенты работают в общем пространстве

Главная ловушка при оркестрации агентов кодинга — два агента, изменяющих одни и те же файлы. Защитные механизмы:

```yaml
delegation:
  git:
    isolate_branches: true            # Каждое делегирование получает свою ветку
    branch_prefix: devin/             # Используйте своё соглашение
    auto_commit: true                 # Коммитить перед возвратом управления
    require_clean_tree: true          # Отказать, если рабочее дерево грязное
  locks:
    strategy: file-level              # Или "workspace" для полной сериализации
```

Hermes создаёт ветку `devin/claude-code-1723487-refactor-auth`, запускает там специалиста, коммитит, возвращает имя ветки и оставляет решение о слиянии вам. Тот же шаблон работает для параллельного делегирования — каждый агент получает свою ветку.

---

## Политика утверждений (Approval Posture)

Агенты кодинга выполняют shell-команды и записывают файлы. Вам нужна политика утверждений, иначе вы потеряете выходные на отладку случайного `rm -rf node_modules` не в той директории.

```yaml
delegation:
  approval:
    default: prompt                   # Запрашивать подтверждение при каждой записи
    trusted_agents:
      - claude-code                   # Наследуют политику утверждений родителя
    auto_approve_read: true           # Инструменты только для чтения никогда не запрашивают подтверждение
    denylist:
      - "rm -rf"
      - "git push --force"
      - "curl * | bash"
```

Полная история в [Части 19](./part19-security-playbook.md#approval-and-denylist-layers). Наследование обхода утверждений появилось в v0.10 ([Часть 16](./part16-backup-debug.md#approval-bypass-for-trusted-subagents)) — используйте его для доверенных специалистов, а не для каждого агента.

---

## Рецепт: Ревью моего PR из Telegram

```
Вы (Telegram): /review_pr myorg/myapp#342
Hermes: *запускает навык `github-pr-review`*
  1. Скачивает diff PR через GitHub MCP
  2. Отправляет в Claude Code с --allowedTools "Read" --max-turns 5
  3. Claude Code возвращает структурированное ревью
  4. Hermes публикует комментарий к PR на GitHub с этим ревью
  5. Отвечает в Telegram сводкой + ссылкой
```

Исходник навыка: `~/.hermes/skills/github-pr-review/SKILL.md` (встроенный, а также варианты, созданные агентом после нескольких использований).

---

## Рецепт: Ночное cron-обслуживание

```yaml
# ~/.hermes/cron.yaml
- name: weekly-dep-audit
  schedule: "0 3 * * 1"                # Понедельники в 3:00
  task: |
    /gemini-cli audit package.json for security advisories
    If any CRITICAL, open a GitHub issue in this repo with the list
  notify: telegram:#engineering
```

Hermes запускает делегирование без присмотра, Gemini с контекстом 1M токенов читает весь lockfile, GitHub MCP открывает issue. Вы просыпаетесь с тикетом для триажа, а не с неожиданной CVE.

---

## Что дальше

- [Часть 17: MCP Servers](./part17-mcp-servers.md) — слой **инструментов (tools)**, которые используют эти агенты кодинга
- [Часть 19: Security Playbook](./part19-security-playbook.md) — защита агентов, выполняющих shell-команды
- [Часть 21: Remote Sandboxes](./part21-remote-sandboxes.md) — запуск агентов кодинга на Modal/Daytona/SSH хосте с вашего телефона
- [Часть 8: Subagent Patterns](./part8-subagent-patterns.md) — базовые примитивы делегирования
