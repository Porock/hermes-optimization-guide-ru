# Участие в разработке

Это руководство создаётся публично. PR приветствуются.

## Что входит в scope

- ✅ Исправления (документация быстро устаревает — функции, цены, номера PR)
- ✅ Новые навыки (skills) в `skills/` (исполняемые файлы `SKILL.md`)
- ✅ Новые шаблоны конфигов в `templates/config/`
- ✅ Новые записи MCP / dashboard / инструментов (tools) в `ECOSYSTEM.md`
- ✅ Вклад в бенчмарки в `benchmarks/` (с заметками о методологии)
- ✅ Новые диаграммы в `diagrams/` (предпочтительно Mermaid)
- ✅ Исправление опечаток, перекрёстных ссылок, форматирования

## Что не входит в scope

- ❌ Маркетинговый контент для коммерческих продуктов (записи в экосистеме должны быть *описательными*, не рекламными)
- ❌ Всё, что опирается на закрытые/недокументированные API Hermes — дождитесь публичного релиза
- ❌ Код или конфиги, напрямую содержащие секреты

## Чек-лист PR

- [ ] Понятный заголовок (префиксы `docs:`, `skill:`, `template:`, `bench:`, `fix:` приветствуются)
- [ ] Для навыков (skills): следуйте структуре из `skills/README.md` (frontmatter, процедура, заметки по безопасности, пример cron если применимо)
- [ ] Для шаблонов: комментируйте каждое неочевидное поле; добавьте заголовок с пояснением, для чего шаблон
- [ ] Для бенчмарков: укажите команду для воспроизведения и дату измерения
- [ ] Никаких секретов, даже в примерах — используйте плейсхолдеры `${VAR}`
- [ ] Перекрёстные ссылки используют относительные пути (`./partN-foo.md`), чтобы они работали в GitHub, VSCode и будущих статических сборках

## Структура репозитория (repository)

```
.
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md                  ← вы здесь
├── ECOSYSTEM.md
├── ROADMAP.md
├── LICENSE
├── part1-setup.md … part23-tenacity-stack.md
├── diagrams/architecture.md
├── skills/
│   ├── README.md
│   ├── security/audit-mcp/SKILL.md
│   ├── security/rotate-secrets/SKILL.md
│   ├── security/audit-approval-bypass/SKILL.md
│   ├── ops/nightly-backup/SKILL.md
│   ├── ops/weekly-dep-audit/SKILL.md
│   ├── ops/cost-report/SKILL.md
│   ├── ops/telegram-triage/SKILL.md
│   ├── dev/pr-review/SKILL.md
│   └── dev/release-notes/SKILL.md
├── templates/
│   ├── config/{minimum,telegram-bot,production,cost-optimized,security-hardened}.yaml
│   ├── compose/langfuse-stack.yml (+ .env example)
│   ├── caddy/Caddyfile
│   ├── systemd/hermes.service + hermes-dashboard.service
│   └── cron/production-crons.yaml
├── scripts/vps-bootstrap.sh
├── benchmarks/README.md + matrix.yaml
└── docs/quickstart.md
```

## Заметки по стилю

- **Простой язык вместо жаргона.** Объясняйте *почему*, а не только *что*.
- **Работоспособность вместо описаний.** Если можете добавить рабочий шаблон или навык (skill) рядом с разделом документации — сделайте это.
- **Подтверждения.** Ссылайтесь на PR, релиз-ноуты, уведомления. Указывайте дату для всего, что устаревает (цены, бенчмарки).
- **Чёткая позиция там, где это важно.** Сказать «Sonnet для кодинга» лучше, чем «вот 7 моделей, выбирайте».

## Локальный просмотр

Подойдёт любой рендерер Markdown. Мы тестируем по рендереру GitHub как источнику истины.

```bash
npx -y prettier --check "**/*.md"          # опционально, мягкая проверка стиля
npx -y markdown-link-check README.md       # валидация перекрёстных ссылок
```

## Кодекс поведения

См. [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md). Если кратко: будьте добры, исходите из добрых намерений, сосредоточьтесь на работе.
