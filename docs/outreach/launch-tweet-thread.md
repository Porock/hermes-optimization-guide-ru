# Запуск треда в Twitter/X — Черновик

**Тон:** по делу, упор на факты, без хайповых формулировок. Замените `@OnlyTerp` / URL репозитория по необходимости.

---

**1/8**
Меня достали гайды по Hermes, которые объясняют архитектуру, но не дают ничего запускаемого, поэтому я сделал наоборот:

24 части документации **плюс** 13 устанавливаемых навыков (skills), 5 продакшн-конфигов, 4 эталонные архитектуры, скрипт развёртывания VPS, ужесточённые systemd-юниты, воспроизводимый бенчмарк расходов и конфигурационный визард (wizard) в браузере.

github.com/OnlyTerp/hermes-optimization-guide

---

**2/8**
5 конфигов: `minimum`, `telegram-bot`, `production`, `cost-optimized`, `security-hardened`.

Каждый — это один `cp` в `~/.hermes/config.yaml`. Они конкретные — не универсальные заготовки — и каждое поле снабжено комментарием.

`templates/config/`

---

**3/8**
Каждый навык (skill), обещанный в гайде — audit-mcp, rotate-secrets, nightly-backup, weekly-dep-audit, cost-report, telegram-triage, pr-review, release-notes, audit-approval-bypass — это настоящий запускаемый файл `SKILL.md`.

```bash
hermes skills install github://OnlyTerp/hermes-optimization-guide/skills/ops/nightly-backup
```

---

**4/8**
Одна команда от свежего Hetzner CX22 до работающего ужесточённого продакшн-решения Hermes:

```bash
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | bash
```

Caddy + UFW + fail2ban + systemd + unattended-upgrades + симлинки навыков (skills). ~10 минут.

---

**5/8**
MCP (Model Context Protocol) стал вирусным на прошлой неделе. В гайде есть целая глава — транспорты stdio/HTTP, 14 серверов, которые стоит установить, `sampling/createMessage`, модель доверия, устранение неполадок.

Директория экосистемы (ECOSYSTEM.md) содержит ссылки на 40+ MCP-серверов, кодинг-агентов (coding agents) и плагинов (plugins) для панелей управления.

---

**6/8**
Атака «Comment and Control» от 15 апреля — межвендорная инъекция промптов (prompt injection), поразившая Claude Code + Gemini CLI + Copilot Agent.

Часть 19 — это руководство по защите: 7 уровней (происхождение, подтверждение, изоляция секретов, подписи вебхуков, SSRF, доверие MCP, карантин). Если ваш агент (agent) читает вашу почту — пожалуйста, прочтите это.

---

**7/8**
Руководство по маршрутизации расходов (Часть 20) снижает типичную нагрузку на ~90%:
- Triage → Gemini Flash или Cerebras
- Классификация → Cerebras Llama (почти бесплатно)
- Кодинг по умолчанию → Kimi/Moonshot
- Сложный кодинг → Sonnet (явное согласие)
- Длинный контекст → Gemini 2.5 Pro

Бенчмарки + методология в `benchmarks/`.

---

**8/8**
Всё под лицензией MIT, `CONTRIBUTING.md` настоящий, CI проверяет frontmatter навыков (skills) + YAML + markdown-ссылки, и есть ROADMAP.

Если это сэкономит вам полдня — звезда поможет большему числу людей найти это. Issues и PRs приветствуются.

github.com/OnlyTerp/hermes-optimization-guide

---

## Ответы / дальнейшие шаги для подготовки

- «Почему не [другой фреймворк]?» → Я не пытаюсь продвигать Hermes; этот гайд был нужен *потому что* мы используем Hermes. Паттерн с конфиг-визардом (config-wizard) и навыками (skills) можно скопировать для любого фреймворка агентов (agent).
- «Работает ли это с локальными моделями?» → Да. Эталонная архитектура `homelab` охватывает маршрутизацию через Ollama. См. `docs/reference-architectures/homelab.md`.
- «Будете ли вы его поддерживать?» → CHANGELOG и ROADMAP — в актуальном состоянии. Фактор автобуса = 1, активно ищу со-мейнтейнеров.
