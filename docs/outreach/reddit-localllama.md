# r/LocalLLaMA — Черновик поста

**Заголовок:** Я выпустил гайд по Hermes с устанавливаемыми навыками (skills), 5 продакшн-конфигами и скриптом для развёртывания VPS одной командой

**Метка (Flair):** `Resources` или `Tutorial | Guide`

---

## Тело поста

r/LocalLLaMA ориентируется на людей, которые **запускают свои собственные системы**, поэтому я пишу с уклоном в `homelab`.

Я создал гайд по оптимизации Hermes (фреймворк для агентов (agent) от Nous Research), который выходит за рамки обычной документации. Всё можно установить — не просто прочитать.

**Репозиторий:** https://github.com/OnlyTerp/hermes-optimization-guide

**Что внутри, что будет полезно этому сабреддиту:**

- **Эталонная архитектура для homelab** — полная настройка для запуска Hermes + LightRAG + самостоятельно размещённого Langfuse на собственном сервере, с Ollama в качестве провайдера (provider) по умолчанию и маршрутизацией только сложных задач на Sonnet. Tailscale вместо проброса портов. Потолки масштабирования + честные компромиссы (задержка, качество и т. д.).

- **5 шаблонов продакшн-конфигов** — один из них `cost-optimized.yaml`, который использует Gemini Flash + Cerebras Llama для основного трафика и переключается на Sonnet только при явном согласии. Типичные расходы: $0.05–0.30/активный час.

- **Воспроизводимые бенчмарки** — 12 флагманских моделей × 5 задач (triage / summarize / codefix / deepreason / bulk-extract), методология + команда `hermes evals run` для воспроизведения.

- **13 устанавливаемых навыков (skill)** (файлы `SKILL.md` с YAML frontmatter — поместите в `~/.hermes/skills/`): audit-mcp, rotate-secrets, audit-approval-bypass, nightly-backup, weekly-dep-audit, cost-report, telegram-triage, pr-review, release-notes, daily-inbox-triage, hermes-weekly, spam-trap, meeting-prep.

- **Гайд по безопасности (Playbook)** (Часть 19) — 7 уровней защиты от инъекций промптов (prompt injection), написан после атаки «Comment and Control» от 15 апреля, поразившей Claude Code + Gemini CLI + Copilot Agent.

- **Глава про MCP** (Часть 17) — транспорты stdio/HTTP, 14 серверов, которые стоит установить уже сейчас, модель доверия, написание собственного сервера за 30 строк.

- **Удалённые песочницы (Remote sandboxes)** (Часть 21) — паттерн «телефон управляет облаком», Modal/Daytona/Fly/E2B. Описан bulk tar-pipe синк из PR от 17 апреля по Hermes.

**Одна команда — от свежего VPS до работающего Hermes:**

```bash
curl -sSL https://raw.githubusercontent.com/OnlyTerp/hermes-optimization-guide/main/scripts/vps-bootstrap.sh | bash
```

Лицензия MIT. CI проверяет frontmatter навыков (skills) + YAML + markdown-ссылки. CHANGELOG и ROADMAP — настоящие.

Если было полезно — звезда поможет большему числу людей найти это. Если что-то не так — открывайте issue или PR.
