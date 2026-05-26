# Часть 8: Паттерны подагентов и оркестратора (Перестаньте делать всё сами)

*Один агент не может делать всё хорошо. Делегируйте.*

---

## Основная идея

Hermes — это оркестратор. Он решает, что делать, затем делегирует выполнение специализированным подагентам. Каждый подагент работает в изоляции — собственный контекст, собственные инструменты, собственная сессия.

**Когда делегировать:**
- Задачи, требующие много reasoning (debugging, code review, research)
- Задачи, которые затопят ваш контекст промежуточными данными
- Параллельные независимые потоки работы (research A и B одновременно)

**Когда НЕ делегировать:**
- Одиночные вызовы инструментов (просто вызовите инструмент напрямую)
- Простые задачи, требующие 1-2 шага
- Задачи, требующие взаимодействия с пользователем (подагенты не могут использовать clarify)

## delegate_task — Основной инструмент

```python
# Одиночная задача
delegate_task(
    goal="Debug why the API returns 403 on POST requests",
    context="File: src/api/client.py. Error started after adding auth headers. Token is valid.",
    toolsets=["terminal", "file"]
)

# Параллельный набор (до 3)
delegate_task(
    tasks=[
        {
            "goal": "Research LightRAG alternatives for graph RAG",
            "toolsets": ["web"]
        },
        {
            "goal": "Benchmark current LightRAG search latency",
            "context="Path: ~/.hermes/skills/research/lightrag/",
            "toolsets": ["terminal"]
        },
        {
            "goal": "Check if our embedding model has a newer version",
            "toolsets": ["web"]
        }
    ]
)
```

**Ключевые детали:**
- Подагенты НЕ имеют памяти вашего разговора. Передавайте всё через `context`.
- Результаты возвращаются как сводка. Промежуточные вызовы инструментов никогда не попадают в ваш контекст.
- Каждый подагент получает собственную терминальную сессию.
- По умолчанию max iterations: 50. Уменьшите для простых задач (`max_iterations=10`).

## Паттерн CEO/COO/Worker

```
CEO (вы + основной агент Hermes)
  │
  ├── COO (delegate_task для планирования/ревью)
  │     └── Возвращает: стратегию, план, заметки ревью
  │
  └── Workers (delegate_task для выполнения)
        ├── Worker 1: Создать фичу A
        ├── Worker 2: Создать фичу B
        └── Worker 3: Написать тесты
```

**CEO:** Принимает решения, назначает задачи, проверяет результаты.
**COO:** Исследует, планирует, ревьювит код. Один подагент, много reasoning.
**Workers:** Выполняют конкретные задачи параллельно. Множество подагентов, много действий.

## ACP подагенты (Claude Code, Codex)

Для задач кодинга делегируйте выделенным кодирующим агентам через ACP:

```python
# Claude Code
delegate_task(
    goal="Implement the user settings page with React",
    context="Repo at /home/terp/my-app. Use existing component library in src/components/",
    acp_command="claude",
    acp_args=["--acp", "--stdio", "--model", "claude-sonnet-5"]
)

# Codex
delegate_task(
    goal="Refactor database layer to use connection pooling",
    context="File: src/db/connection.py. Currently opens new connection per query.",
    acp_command="codex"
)
```

**Когда использовать ACP vs обычный delegate_task:**
- ACP агенты (Claude Code, Codex) лучше в кодинге — tool calling, редактирование файлов, запуск тестов
- Обычный delegate_task лучше для research, анализа и многоинструментных рабочих процессов
- ACP агенты быстрее для редактирования одиночных файлов

## SWE-1.6 через Windsurf Cascade

Для сложных задач кодинга используйте Windsurf's SWE-1.6:

```python
# Отправить задачу кодинга в Windsurf Cascade
# Требует Windsurf запущенный с --remote-debugging-port=9222
subprocess.run([
    "python",
    "~/.hermes/skills/autonomous-ai-agents/windsurf-cascade/scripts/cascade_send.py",
    "Build a React dashboard with real-time WebSocket updates"
])
```

**Паттерн оркестратора:** Hermes обрабатывает APIs, данные, решения. SWE-1.6 обрабатывает UI, компоненты, исправления багов. Каждый делает то, что лучше всего умеет.

## Правила параллелизации

| Сценарий | Подход |
|----------|----------|
| 3 независимые задачи research | Batch `delegate_task` с массивом `tasks` |
| 1 сложная задача кодинга | ACP подагент (Claude Code или Codex) |
| Множественные изменения кода в разных файлах | SWE-1.6 через Cascade |
| Одиночный API вызов | Просто вызовите инструмент, не делегируйте |
| Задача требует ввода пользователя | Сделайте сами, нельзя делегировать интерактивную работу |

## Типичные ошибки

| Ошибка | Исправление |
|---------|-----|
| Делегирование одиночного вызова инструмента | Просто вызовите инструмент напрямую |
| Недостаточно контекста подагенту | Подагенты ничего не знают — передавайте пути к файлам, сообщения об ошибках, ограничения |
| Делегирование последовательных задач параллельно | Если задача B зависит от вывода задачи A, запустите их последовательно |
| Слишком высокий max_iterations | Простым задачам не нужно 50 итераций — используйте 10-15 |
| Забывание, что подагенты не могут использовать clarify | Если задача может потребовать уточнения, сделайте сами |

---

## Что дальше (дополнения апреля 2026)

Система подагентов быстро выросла. Продолжайте с:

- **[Часть 18: Делегирование кодирующим агентам](./part18-coding-agents.md)** — паттерн OpenClaw (thread-bound Telegram topics → persistent Claude Code / Codex / Gemini CLI runtimes). Print-mode vs interactive, ACP-as-server, изоляция git branch, правила роутинга.
- **[Часть 17: MCP-серверы](./part17-mcp-servers.md)** — дайте подагентам инструменты, которые синхронизируются между Hermes, Claude Code и Cursor.
- **[Часть 21: Удалённые песочницы](./part21-remote-sandboxes.md)** — запускайте ваших подагентов на Modal/Daytona/SSH чтобы $5 VPS мог управлять мощным workspace.
- **[Часть 20: Observability](./part20-observability.md)** — трассируйте каждый вызов подагента в Langfuse с разбивкой стоимости по навыкам.

---

*Паттерн оркестратора — как вы масштабируетесь. Один мозг, много рук.*