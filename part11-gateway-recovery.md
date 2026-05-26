# Часть 11: Восстановление Шлюза (Когда Что-то Ломается в 3 Ночи)

*Шлюз — это мозговой ствол. Когда он падает, всё останавливается.*

---

## Что Делает Шлюз

Шлюз (`hermes gateway`) — это всегда-running процесс, который:
- Получает сообщения из Telegram, Discord, Slack, CLI
- Маршрутизирует их к агенту
- Управляет сессиями и контекстом
- Запускает cron задачи

Если шлюз умирает, ваш агент недоступен.

## Обнаружение Сбоя

```bash
# Проверить, работает ли шлюз
hermes status

# Или напрямую
ps aux | grep hermes-gateway

# Проверить логи
tail -50 ~/.hermes/logs/gateway.log
```

## Типичные Причины Сбоев

### 1. Переполнение Контекстного Окна

**Симптомы:** Шлюз умирает посреди ответа, логи показывают ошибки подсчёта токенов.

**Исправление:** Уменьшите контекстную инъекцию в `~/.hermes/.env`:

```bash
# Уменьшите max контекст (по умолчанию обычно max модели)
MAX_CONTEXT_TOKENS=80000

# Включите сжатие раньше
CONTEXT_COMPRESSION_THRESHOLD=70
```

### 2. OOM (Недостаток Памяти)

**Симптомы:** Шлюз убит OOM killer, `dmesg` показывает `Out of memory: Killed process`.

**Исправление:**

```bash
# Проверить использование памяти
free -h

# Если используете локальные модели через Ollama, они едят VRAM/RAM
# Переместите Ollama на другую машину или уменьшите размер модели

# Ограничьте память шлюза
# В systemd сервисе или скрипте запуска:
systemctl edit hermes-gateway
# Добавить: MemoryMax=4G
```

### 3. API Провайдер Упал

**Симптомы:** Шлюз работает, но все ответы падают, логи показывают ошибки подключения.

**Исправление:** Настройте fallback провайдеров (см. Часть 9):

```yaml
model_fallback:
  - provider: cerebras
    model: llama-3.3-70b
  - provider: openrouter
    model: anthropic/claude-sonnet-5
  - provider: local
    model: nemotron:latest
```

### 4. Диск Полный

**Симптомы:** Шлюз не может записать файлы сессий, логи или базу данных памяти.

**Исправление:**

```bash
# Проверить место на диске
df -h

# Очистить старые файлы сессий (безопасно удалять)
find ~/.hermes/sessions -mtime +30 -delete

# Очистить старые логи
find ~/.hermes/logs -mtime +7 -delete

# Проверить размер данных LightRAG
du -sh ~/.hermes/skills/research/lightrag/data/
```

### 5. Цикл Сбоев

**Симптомы:** Шлюз запускается, сразу падает, повторяется.

**Исправление:**

```bash
# Проверить последний лог сбоя
tail -100 ~/.hermes/logs/gateway.log

# Частая причина: повреждённый файл сессии
# Временно переместите сессии
mv ~/.hermes/sessions ~/.hermes/sessions.bak
mkdir ~/.hermes/sessions

# Перезапустите
hermes gateway

# Если работает, проблема была в повреждённой сессии
# Перемещайте сессии обратно по одной, чтобы найти плохую
```

## Авто-Восстановление (systemd)

Настройте systemd для авто-перезапуска шлюза:

```ini
# /etc/systemd/system/hermes-gateway.service
[Unit]
Description=Hermes Agent Gateway
After=network.target

[Service]
Type=simple
User=terp
WorkingDirectory=/home/terp/.hermes
ExecStart=/home/terp/.hermes/venv/bin/python -m hermes_gateway
Restart=always
RestartSec=5
MemoryMax=4G

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable hermes-gateway
sudo systemctl start hermes-gateway

# Проверить статус
sudo systemctl status hermes-gateway

# Смотреть логи
journalctl -u hermes-gateway -f
```

## Авто-Восстановление (Cron Fallback)

Если не можете использовать systemd, используйте cron watchdog:

```bash
# Добавить в crontab -e
* * * * * pgrep -f "hermes.*gateway" > /dev/null || (cd ~/.hermes && nohup ./venv/bin/python -m hermes_gateway >> logs/watchdog.log 2>&1 &)
```

Проверяет каждую минуту. Если шлюз не работает, запускает его.

## Проверка Здоровья

Быстрый скрипт для проверки, что всё работает:

```bash
#!/bin/bash
# ~/.hermes/scripts/health-check.sh

# Шлюз работает?
if ! pgrep -f "hermes.*gateway" > /dev/null; then
    echo "CRITICAL: Gateway not running"
    exit 1
fi

# Можем ли мы достучаться до API?
if ! curl -s http://localhost:8642/health > /dev/null 2>&1; then
    echo "CRITICAL: Gateway not responding"
    exit 1
fi

# Место на диске OK?
USAGE=$(df -h ~/.hermes | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$USAGE" -gt 90 ]; then
    echo "WARNING: Disk usage at ${USAGE}%"
    exit 1
fi

echo "OK"
```

---

*Шлюз должен быть скучным. Если он интересен, что-то не так.*