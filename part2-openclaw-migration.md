# Часть 2: Миграция (Migration) из OpenClaw (не оставляйте свои знания позади)

*Перенесите свои навыки (skills), память (memory), конфигурацию и личность из OpenClaw в Hermes одной командой.*

---

## Зачем мигрировать

Hermes — преемник OpenClaw. Если вы потратили недели или месяцы на создание навыков (skills), файлов памяти (memory) и конфигурации в OpenClaw, инструмент миграции (migration) перенесёт всё это автоматически.

**Что переносится (transfers):**

| Что | Расположение в OpenClaw | Назначение в Hermes |
|------|------------------|-------------------|
| Личность | `workspace/SOUL.md` | `~/.hermes/SOUL.md` |
| Инструкции | `workspace/AGENTS.md` | Указанная директория workspace |
| Память (Memory) | `workspace/MEMORY.md` + `workspace/memory/*.md` | `~/.hermes/memories/MEMORY.md` (объединено, без дубликатов) |
| Профиль пользователя | `workspace/USER.md` | `~/.hermes/memories/USER.md` |
| Навыки (Skills) | `workspace/skills/`, `~/.openclaw/skills/` | `~/.hermes/skills/openclaw-imports/` |
| Конфигурация модели | `agents.defaults.model` | `config.yaml` |
| Ключи провайдеров (providers) | `models.providers.*.apiKey` | `~/.hermes/.env` (с `--migrate-secrets`) |
| Кастомные провайдеры (providers) | `models.providers.*` | `config.yaml → custom_providers` |
| Максимум ходов | `agents.defaults.timeoutSeconds` | `agent.max_turns` (timeoutSeconds / 10) |

> **Примечание:** Транскрипты сессий (sessions), определения cron-задач и данные плагинов (plugins) не переносятся. Они специфичны для OpenClaw и имеют другие форматы в Hermes.

---

## Быстрая миграция (Migration)

```bash
# Предпросмотр того, что произойдёт (файлы не изменяются)
hermes claw migrate --dry-run

# Запуск полной миграции (включая API-ключи)
hermes claw migrate

# Исключить API-ключи (безопаснее для общих машин)
hermes claw migrate --preset user-data
```

По умолчанию миграция (migration) читает данные из `~/.openclaw/`. Если у вас есть устаревшие директории `~/.clawdbot/` или `~/.moldbot/`, они обнаруживаются автоматически.

---

## Параметры миграции (Migration Options)

| Параметр | Что делает | По умолчанию |
|--------|-------------|---------|
| `--dry-run` | Предпросмотр без записи | off |
| `--preset full` | Включает API-ключи и секреты | yes |
| `--preset user-data` | Исключает API-ключи | no |
| `--overwrite` | Перезаписывает существующие файлы Hermes при конфликтах | skip |
| `--migrate-secrets` | Явно включает API-ключи | on с `--preset full` |
| `--source <path>` | Кастомная директория OpenClaw | `~/.openclaw/` |
| `--workspace-target <path>` | Куда поместить `AGENTS.md` | текущая директория |
| `--skill-conflict <mode>` | `skip`, `overwrite` или `rename` | `skip` |
| `--yes` | Пропустить подтверждение | off |

---

## Пошаговое руководство

### 1. Сначала сухой прогон (Dry Run)

Всегда просматривайте предпросмотр перед применением:

```bash
hermes claw migrate --dry-run
```

Это покажет, какие файлы будут созданы, перезаписаны или пропущены. Внимательно изучите вывод.

### 2. Запустите миграцию (Migration)

```bash
hermes claw migrate
```

Инструмент (tool) выполнит:
1. Обнаружение вашей установки OpenClaw
2. Сопоставление ключей конфигурации с аналогами в Hermes
3. Объединение файлов памяти (memory) (дедупликация записей)
4. Копирование навыков (skills) в `~/.hermes/skills/openclaw-imports/`
5. Перенос API-ключей (если `--preset full`)
6. Отчёт о выполненной работе

### 3. Обработка конфликтов

Если навык (skill) с таким же именем уже существует в Hermes:

- **`--skill-conflict skip`** (по умолчанию): Оставляет версию Hermes, пропускает импорт
- **`--skill-conflict overwrite`**: Заменяет версию Hermes версией из OpenClaw
- **`--skill-conflict rename`**: Создаёт копию с суффиксом `-imported` рядом с версией Hermes

```bash
# Пример: переименовать при конфликте, чтобы можно было сравнить
hermes claw migrate --skill-conflict rename
```

### 4. Проверка после миграции (Migration)

```bash
# Проверьте, загрузилась ли личность
cat ~/.hermes/SOUL.md

# Проверьте, объединились ли записи памяти (memory)
cat ~/.hermes/memories/MEMORY.md | head -50

# Проверьте импортированные навыки (skills)
ls ~/.hermes/skills/openclaw-imports/

# Протестируйте агента (agent)
hermes chat -q "Что ты обо мне помнишь?"
```

---

## Что не переносится

| Элемент | Почему | Что делать |
|------|-----|-----------|
| Транскрипты сессий (sessions) | Другой формат | Заархивировать вручную при необходимости |
| Определения cron-задач | Другой планировщик | Создать заново через `hermes cron` |
| Конфигурации плагинов (plugins) | Система плагинов (plugins) изменилась | Перенастроить в Hermes |
| Функции, специфичные для OpenClaw | Могут ещё отсутствовать | Проверьте документацию Hermes на наличие аналогов |

---

## Сопоставление ключей конфигурации (Config Key Mapping)

Для справки — как конфигурация OpenClaw сопоставляется с Hermes:

| OpenClaw Config | Hermes Config | Примечания |
|----------------|---------------|-------|
| `agents.defaults.model` | `model` | Строка или `{primary, fallbacks}` |
| `agents.defaults.timeoutSeconds` | `agent.max_turns` | Делится на 10, максимум 200 |
| `agents.defaults.verboseDefault` | `agent.verbose` | off / on / full |
| `agents.defaults.thinkingDefault` | `reasoning.mode` | off / low / high |
| `models.providers.*.baseUrl` | `custom_providers.*.base_url` | Прямое сопоставление |
| `models.providers.*.apiType` | `custom_providers.*.api_type` | openai → chat_completions, anthropic → anthropic_messages |

---

## Устранение неполадок (Troubleshooting)

### «Установка OpenClaw не найдена»

Убедитесь, что данные OpenClaw находятся в `~/.openclaw/`. Если они в другом месте:

```bash
hermes claw migrate --source /путь/к/вашему/openclaw
```

### Записи памяти (memory) выглядят дублированными

Миграция (migration) дедуплицирует по схожести содержимого, но если в памяти OpenClaw были почти дубликаты, они могут объединиться неидеально. Очистите вручную:

```bash
# Отредактируйте память (memory) напрямую
nano ~/.hermes/memories/MEMORY.md
```

### Ошибки импорта навыков (skills)

Навыки (skills) OpenClaw могут ссылаться на модули или шаблоны, которых нет в Hermes. Откройте файл навыка (skill) и проверьте импорты:

```bash
cat ~/.hermes/skills/openclaw-imports/имя-навыка/SKILL.md
```

Большинство навыков (skills) работают как есть, поскольку это инструкции в формате Markdown. Навыки (skills) с кодом, импортирующим специфичные для OpenClaw модули, требуют ручного обновления.

---

## Что дальше

- **Хотите более умную память (memory)?** → [Часть 3: Настройка LightRAG](./part3-lightrag-setup.md)
- **Нужен мобильный доступ?** → [Часть 4: Настройка Telegram](./part4-telegram-setup.md)
- **Хотите, чтобы агент (agent) саморазвивался?** → [Часть 5: Навыки (Skills) на лету](./part5-creating-skills.md)
