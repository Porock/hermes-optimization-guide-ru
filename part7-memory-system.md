# Часть 7: Система памяти (Три уровня, которые действительно работают)

*У Hermes три системы памяти. Большинство знает только об одной.*

---

## Три уровня

| Инструмент | Что делает | Когда срабатывает | Стоимость |
|------|-------------|---------------|------|
| `memory` | Постоянные факты во всех сессиях | Предпочтения пользователя, окружение, уроки | Бесплатно (локально) |
| `session_search` | Поиск в прошлых транскриптах разговоров | "Что мы решили насчёт X?" или "Помнишь, когда..." | Бесплатно (локально) |
| `skill_manage` | Процедурная память — переиспользуемые рабочие процессы | После исправления бага, построения чего-то сложного или обнаружения нового подхода | Бесплатно (локально) |

Все три — **local-first**. Никаких API вызовов, никаких затрат на эмбеддинги. Они используют SQLite и полнотекстовый поиск.

## Уровень 1: memory (Постоянные факты)

Инструмент `memory` сохраняет долговечные факты, которые внедряются в каждую будущую сессию.

**Что сохранять:**
- Предпочтения пользователя ("Терп ненавидит ручные шаги")
- Детали окружения ("5090 PC на 192.168.1.67, порт 11434")
- Особенности инструментов ("PowerShell нужен -Encoding utf8 для Unicode файлов")
- Стабильные конвенции ("Используй OnlyTerp для GitHub репозиториев")

**Что НЕ сохранять:**
- Прогресс задачи (используйте session_search для восстановления)
- Временное состояние (TODO списки, текущий статус)
- Что-либо, что меняется часто

**Формат:** Держите записи под 2000 символов. Будьте компактны. Они внедряются в каждое сообщение.

```python
# Хорошо
memory(action="add", target="memory", content="OpenClaw мигрирован. LightRAG: 4528 сущностей, float16 векторы (4096d). Telegram бот 8624585264, группа -5216536760.")

# Плохо — слишком многословно, специфично для задачи
memory(action="add", target="memory", content="Сегодня я работал над воронкой лидов. Сначала я исправил проблему с API ключом, потом я обновил scoring quality gate на новый алгоритм, потом я протестировал с 50 лидами...")
```

## Уровень 2: session_search (Воспоминания разговора)

`session_search` ищет по всей вашей истории разговоров во всех прошлых сессиях.

**Два режима:**

```python
# Просмотр недавних сессий (бесплатно, мгновенно)
session_search()

# Поиск по конкретным темам (использует LLM для суммирования)
session_search(query="hermes optimization guide github")
session_search(query="LightRAG setup OR embedding model")
```

**Когда использовать:**
- Пользователь говорит "мы это делали раньше" или "помнишь, когда"
- Вы подозреваете, что существует релевантный кросс-сессионный контекст
- Вы хотите проверить, решали ли вы подобную проблему раньше

**Ключевой инсайт:** session_search — ваш backup на недавнее. memory — для фактов, которые будут важны через 6 месяцев. Если факт релевантен только для текущей фазы проекта, session_search лучше, чем раздувать память.

## Уровень 3: skill_manage (Процедурная память)

`skill_manage` сохраняет переиспользуемые рабочие процессы как навыки (skills). Так Hermes учится.

**Когда создавать навык:**
- После сложной задачи (5+ вызовов инструментов)
- После исправления хитрой ошибки
- После обнаружения нетривиального рабочего процесса
- Когда пользователь просит вас запомнить процедуру

```python
# Создать новый навык
skill_manage(
    action="create",
    name="supabase-migrate",
    content="---\ndescription: Run Supabase SQL migrations via Management API\n---\n\n# Supabase Migration\n\n1. Read the SQL file from supabase/migrations/\n2. Use Python http.client to POST to Management API...",
    category="devops"
)

# Patch an existing skill when you find issues
skill_manage(
    action="patch",
    name="supabase-migrate",
    old_string="Use requests.post",
    new_string="Use http.client (requests has timeout issues with Supabase)"
)
```

**Key rules:**
- Skills must have trigger conditions — when should this skill load?
- Skills must have numbered steps — what exactly to do?
- Skills must have pitfalls — what can go wrong?
- Patch skills immediately when you find issues — don't wait to be asked

## How They Work Together

```
User asks a question
    ↓
memory injects persistent context (user prefs, environment)
    ↓
session_search recalls relevant past conversations (if needed)
    ↓
skill_manage loads procedural knowledge (if triggered)
    ↓
Agent has full context → better answer
```

**The hierarchy:** memory is always on. session_search is on-demand. skill_manage is triggered by task matching.

## Anti-Patterns

| Don't Do This | Do This Instead |
|--------------|-----------------|
| Save task progress to memory | Use session_search to recall |
| Create a skill for a one-off task | Just do it, skip the skill |
| Dump raw data into memory | Save compact, durable facts |
| Search session_search for everything | Check memory first, it's free and instant |
| Let skills go stale | Patch them immediately when outdated |

---

*Memory is what separates a stateless chatbot from an actual agent. Use all three tiers.*
