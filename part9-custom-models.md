# Часть 9: Кастомные провайдеры моделей (Используйте любую модель)

*Hermes поддерживает любой OpenAI-совместимый API, плюс первоклассные нативные адаптеры для Nous Portal, Anthropic, OpenAI/Codex, OpenRouter, AWS Bedrock, Azure AI Foundry, Google Gemini, Gemini OAuth, LM Studio, xAI, Xiaomi MiMo, Kimi/Moonshot, z.ai/GLM, MiniMax, Arcee, GMI Cloud, Tencent TokenHub, Hugging Face, Cerebras, Groq, Fireworks, Vercel AI Gateway, Ollama и плагины провайдеров. Это шпаргалка от 14 мая 2026 года.*

> **Что нового с обновления руководства v0.12** — v0.13 делает провайдеров подключаемыми, добавляет роутинг с учётом медиа типа `video_analyze`, улучшает обработку медиа в MCP, держит Gemini OAuth внутри `hermes model`, и делает выборщики моделей OpenRouter/Nous/Vercel зависимыми от live-манифестов вместо захардкоженных снапшотов релизов.

---

## Нативные адаптеры против.generic OpenAI-совместимых

По состоянию на v0.13.0 (май 2026), Hermes поставляется с **нативными адаптерами** для большого набора провайдеров, плюс поверхность плагинов провайдеров для бэкендов вне дерева. Нативные адаптеры знают о специфичных для провайдера функциях, которые generic OpenAI-совместимый враппер не может:

| Провайдер | Нативный адаптер? | Примечательная функция |
|----------|-----------------|-----------------|
| **Nous Portal** | Да | Auth через `hermes model` (без голого API ключа). Разблокирует [Tool Gateway](./part13-tool-gateway.md). |
| **Anthropic** | Да | Нативное кэширование промптов, extended thinking, `/fast` приоритет |
| **OpenAI** | Да | Нативный responses API, уровни reasoning effort, `/fast` приоритет |
| **OpenAI Codex OAuth** | Да | ChatGPT/Codex логин через `hermes model`, без API ключа |
| **AWS Bedrock** | Да | Converse API, IAM credentials, cross-region inference profiles, Bedrock Guardrails |
| **Azure AI Foundry** | Да | Автоопределяет OpenAI-style vs Anthropic-style деплойменты и длину контекста |
| **LM Studio** | Да | Локальное обнаружение `/models`, опциональный auth, reasoning transport, проверки `hermes doctor` |
| **xAI (Grok)** | Да | Нативный live X поиск и интеграции xAI image/STT/TTS, включая Custom Voices |
| **Xiaomi MiMo** | Да | Нативные режимы reasoning (`low`/`medium`/`high`) доступные как конфиг |
| **Kimi / Moonshot** | Да | 200K+ контекст, отлично подходит для извлечения сущностей LightRAG (см. [Часть 3](./README.md#part-3-lightrag--graph-rag-that-actually-works)) |
| **z.ai / GLM** | Да | Сильные open-weight модели для использования инструментов; хороший дешёвый fallback для планирования/исследования |
| **Google Gemini (direct)** | Да | 1M контекст; нативное кэширование промптов на Pro; роутинг моделей с поддержкой image/video |
| **Google Gemini (OAuth)** | Да | Browser PKCE логин через `hermes model`; бесплатный tier поддерживается; внешняя установка `gemini` не нужна |
| **MiniMax** | Да | API ключ или OAuth; нативный streaming и TTS |
| **GMI Cloud** | Да | Хостинг open моделей через нативный провайдер |
| **Tencent TokenHub** | Да | Роутинг моделей Tencent через TokenHub алиасы |
| **Arcee** | Да | AFM-4.5 специалист по function-calling, дёшево |
| **Cerebras** | Да | 2000+ tok/s инференс |
| **Groq** | Да | Быстрый хостинг Llama / Qwen |
| **Fireworks** | Да | Qwen3-Embedding-8B (рекомендуется для LightRAG) |
| **Vercel AI Gateway** | Да | Динамическое обнаружение моделей, метаданные ценообразования, атрибуция |
| **Hugging Face** | Да | Любой TGI / TEI endpoint (self-hosted или Inference Endpoints) |
| **OpenRouter** | Да | Pass-through к 200+ моделям; уважает особенности нативного адаптера, когда downstream — это он |
| **Ollama** (локально) | Generic | OpenAI-совместимый, нулевой auth |
| **Плагин провайдера** | Плагин | Вставьте `ProviderProfile` без патчинга ядра Hermes |
| **Всё остальное** | Generic | Любой OpenAI-совместимый `base_url` |

Выбирайте нативный адаптер, когда он существует — вы получаете специфичные для провайдера функции бесплатно. Используйте generic OpenAI-совместимый путь только для эндпоинтов, у которых ещё нет нативного адаптера.

### Шпаргалка по провайдерам (14 мая 2026)

Точная "лучшая модель" меняется еженедельно, поэтому воспринимайте это как позицию роутинга, а не лидерборд. Используйте `hermes model` для live данных выбора, затем закрепите только то, что вам нужно воспроизводимым.

| Потребность | Начните здесь | Почему |
|------|-------------|-----|
| Кодинг по умолчанию / рефакторинг | Anthropic Sonnet 5, Claude Code, или Codex OAuth | Лучшая надёжность для работы с множеством патчей; Codex OAuth избегает API-key churn |
| Глубокий reasoning / высокие ставки | GPT-5.5 reasoning или Anthropic Opus 4.7 | Используйте явно; не делайте по умолчанию для cron/массовых задач |
| Длинный контекст чтение репо или документов | Gemini 3.1 Pro/Flash или эквивалент OpenRouter | Огромное окно, достаточно дёшево для map/reduce, video и суммирования |
| Дешёвый ежедневный драйвер | Gemini OAuth + Kimi K2.6 + z.ai/GLM | Хорошее соотношение качество/стоимость, особенно с auxiliary роутингом |
| Enterprise / VPC / compliance | AWS Bedrock или Azure AI Foundry | IAM/Azure auth, guardrails, private деплойменты, аудит контроли |
| Локально/приватно/офлайн | LM Studio или Ollama | Нет облачного egress; отлично для извлечения, эмбеддингов и черновиков |
| Сверхбыстрые интерактивные ходы | Cerebras или Groq | Очень высокие tok/s; полезно для классификации и коротких чатов |
| Поиск текущих событий | xAI Grok 4.x или веб-поиск через инструменты | Grok имеет нативный live-X поиск; Tool Gateway может покрыть более широкий веб |

> Ценообразование и контекстные окна меняются слишком быстро, чтобы хардкодить. Hermes теперь тянет списки выбора моделей OpenRouter и Nous Portal из удалённого манифеста, а провайдеры API предоставляют метаданные ценообразования/контекста там, где доступно.

---

### Nous Portal — OAuth, Не API Ключ

Nous Portal использует OAuth поток через `hermes model` вместо голого API ключа. После auth учётные данные живут в `~/.hermes/auth.json` (никогда в `.env`). Re-auth когда истекает:

```bash
hermes model
# Выберите "Nous Portal" → завершите browser OAuth поток
```

Если у вас платная подписка, настройка также предлагает включить [Tool Gateway](./part13-tool-gateway.md) — веб-поиск, генерация изображений, TTS и browser automation через вашу подписку, без дополнительных ключей.

### Gemini OAuth — Для Бесплатного Тира

Если у вас есть Google аккаунт, полностью пропустите API ключ и войдите из Hermes:

```bash
hermes model
# Выберите "Google Gemini (OAuth)" → завершите browser PKCE поток
```

Токены хранятся в `~/.hermes/auth/google_oauth.json` с правами 0600 и автоматическим обновлением. На headless SSH боксах Hermes падает обратно на paste-mode auth.

### AWS Bedrock и Azure AI Foundry — Роутинг Для Предприятий Без Прокси

Bedrock использует нативный Converse API и обычную boto3 цепочку учётных данных:

```bash
pip install 'hermes-agent[bedrock]'
hermes model
# Выберите "AWS Bedrock" → region → model/profile
```

Используйте это, когда хотите IAM роли, Bedrock Guardrails и cross-region inference профили вместо прямых API ключей вендора.

Azure AI Foundry обрабатывает оба стиля эндпоинтов:

```bash
hermes model
# Выберите "Azure Foundry" → вставьте endpoint + key
```

Hermes зондирует эндпоинт, определяет OpenAI-style `/chat/completions` vs Anthropic-style `/messages`, обнаруживает деплойменты когда возможно, и хранит правильный `api_mode` в `config.yaml`.

### Удалённый Каталог Моделей: Перестаньте Хардкодить Победителя Этой Недели

Выборщики моделей OpenRouter и Nous Portal теперь получают:

```text
https://hermes-agent.nousresearch.com/docs/api/model-catalog.json
```

Кэш живёт в `~/.hermes/cache/model_catalog.json`. Если манифест недоступен, Hermes падает на дисковый кэш или bundled снапшот, так что выбор моделей всё ещё работает офлайн.

### Gemini TTS

Gemini теперь один из практических voice бэкендов наряду с Edge, ElevenLabs, OpenAI, MiniMax, Mistral, NeuTTS и xAI:

```yaml
tts:
  gemini:
    model: gemini-2.5-flash-preview-tts
    voice: Kore
```

`GEMINI_API_KEY` или `GOOGLE_API_KEY` достаточно. Вывод приходит как PCM, обёрнутый в WAV нативно (без доп. зависимостей), опционально конвертируется в mp3/ogg через `ffmpeg`. Работает для Telegram voice bubbles из коробки.

---

## Структура config.yaml

Модели настраиваются в `~/.hermes/config.yaml`:

> **Заметка по безопасности:** Никогда не кладите реальные API ключи напрямую в `config.yaml`. Используйте ссылки на переменные окружения, чтобы ключи оставались в `~/.hermes/.env` (который должен быть `chmod 600` и никогда не коммититься в git).

```yaml
# Модель по умолчанию
model: claude-sonnet
provider: anthropic

# Конфигурации провайдеров
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}

  openai:
    api_key: ${OPENAI_API_KEY}

  bedrock:
    region: us-east-2                  # Auth через AWS_PROFILE, env vars или instance role

  azure-foundry:
    api_key: ${AZURE_FOUNDRY_API_KEY}
    base_url: ${AZURE_FOUNDRY_ENDPOINT}
    api_mode: chat_completions         # Или anthropic_messages; wizard автоопределяет

  lmstudio:
    base_url: http://127.0.0.1:1234/v1
    api_key: ${LM_API_KEY}             # Опционально, если ваш LM Studio сервер требует auth

  xai:
    api_key: ${XAI_API_KEY}
    live_search: true                 # Grok's live X/Twitter поиск

  xiaomi:
    api_key: ${XIAOMI_API_KEY}
    reasoning_mode: high              # low / medium / high

  moonshot:                           # Kimi
    api_key: ${MOONSHOT_API_KEY}

  zai:                                # z.ai / GLM
    api_key: ${ZAI_API_KEY}

  minimax:
    api_key: ${MINIMAX_API_KEY}

  gmi:
    api_key: ${GMI_API_KEY}

  tencent-tokenhub:
    api_key: ${TOKENHUB_API_KEY}

  arcee:
    api_key: ${ARCEE_API_KEY}

  cerebras:
    api_key: ${CEREBRAS_API_KEY}
    base_url: https://api.cerebras.ai/v1

  fireworks:
    api_key: ${FIREWORKS_API_KEY}
    base_url: https://api.fireworks.ai/inference/v1

  local:
    base_url: http://localhost:11434/v1
    api_key: ollama  # Ollama не требует реального ключа
```

## Добавление Кастомного Провайдера

Любой провайдер, реализующий OpenAI chat completions API, работает:

```yaml
providers:
  my-custom:
    api_key: ${MY_CUSTOM_API_KEY}
    base_url: https://api.your-provider.com/v1
```

Добавьте реальный ключ в ваш файл `.env`:

```bash
echo "MY_CUSTOM_API_KEY=<your-key-here>" >> ~/.hermes/.env
chmod 600 ~/.hermes/.env
```

Затем используйте:

```bash
hermes --provider my-custom --model their-model-name
```

## Алиасы Моделей (Быстрое Переключение)

Добавьте алиасы для переключения моделей без ввода полных имён:

```yaml
model_aliases:
  fast:
    model: cerebras/llama-3.3-70b
    provider: cerebras
  smart:
    model: claude-opus-4.7
    provider: anthropic
  local:
    model: nemotron:latest
    provider: local
```

Используйте в чате:

```
/model fast      # Переключить на Cerebras Llama 70B
/model smart     # Переключить на Claude Opus
/model local     # Переключить на локальную модель Ollama
```

## Сравнение Провайдеров (Что Мы Действительно Используем)

| Провайдер | Скорость | Стоимость | Лучше всего для |
|----------|-------|------|----------|
| Cerebras | 3000+ tok/s | Дёшево | Быстрый инференс, массовые задачи, кодинг |
| Anthropic | ~100 tok/s | Премиум | Сложный reasoning, длинный контекст |
| OpenRouter | Варьируется | Варьируется | Разнообразие моделей, fallback провайдер |
| Fireworks | Быстро | Дёшево | Эмбеддинги, специализированные модели |
| Ollama (локально) | Варьируется | Бесплатно | Приватность, офлайн, эксперименты |

**Наша настройка:** Cerebras для скорости, Anthropic для качества, Ollama для локальных моделей и эмбеддингов.

## Шпаргалка По Роутингу По Типу Задачи

Используйте это как авторские умолчания, затем настраивайте с помощью [роутинга стоимости из Части 20](./part20-observability.md#cost-routing-playbook-the-one-that-actually-saves-money):

| Задача | Первый выбор | Fallback (дешевле) | Fallback (быстрее) |
|------|--------------|--------------------|--------------------|
| Ежедневный разговор | Anthropic Sonnet 5 | Gemini OAuth или z.ai/GLM | Cerebras Llama/Qwen |
| Делегирование кодинга | Claude Code / Codex OAuth | OpenCode + Kimi K2.6 | OpenCode + Cerebras |
| Длинный контекст чтение (>200K) | Gemini 3.1 Pro | Gemini Flash | — |
| Классификация / сортировка | Gemini Flash | Cerebras Qwen3 32B | Arcee AFM-4.5 |
| Reasoning (математика, планирование) | GPT-5.5 reasoning | Anthropic Opus 4.7 | z.ai/GLM |
| Текущие события / live поиск | xAI Grok 4.x | Gemini с grounding | Tool Gateway веб-поиск |
| Эмбеддинги (LightRAG) | Qwen3-Embedding-8B (Fireworks) | nomic-embed-text (Ollama) | OpenAI `text-embedding-3-small` |
| TTS (голос в Telegram) | xAI Custom Voices или Tool Gateway TTS | Gemini Flash TTS | Edge TTS (бесплатно) |
| Vision / video | Gemini 3.1 Pro/Flash | GPT-5.5 мультимодальный | Claude Sonnet 5 |

---

## Подводные камни Cerebras

Cerebras быстрый, но есть особенности:

1. **Нет кэширования системного промпта.** Каждый запрос заново отправляет полный системный промпт. Держите его коротким.
2. **Rate limits в минуту, не за запрос.** Батчите аккуратно.
3. **Некоторые модели не поддерживают tool calling.** Проверьте перед использованием как основную модель агента.
4. **Стриминг быстрый, но чанки.** Большие ответы приходят большими всплесками, не плавными потоками.

Конфиг:

```yaml
providers:
  cerebras:
    api_key: ${CEREBRAS_API_KEY}
    base_url: https://api.cerebras.ai/v1
    # Models: llama-3.3-70b, llama-4-scout-17b-16e-instruct, qwen-3-32b
```

## Локальные Модели (Ollama)

Запускайте модели локально для бесплатного инференса:

```yaml
providers:
  local:
    base_url: http://localhost:11434/v1
    api_key: ollama
```

**Лучшие локальные/открытые модели для Hermes:**
- **Qwen3-Coder-Next** — сильнейшая локальная кодовая дорожка, если у вас 24GB+ VRAM
- **DeepSeek V4-Flash / V4-Pro** — сильный open-weight reasoning/кодинг, если можете удобно хостить MoE
- **Qwen3.6-27B / 32B** — практичный баланс reasoning/кодинг на одной рабочей станции
- **Nemotron 30B** — хороший универсальный fallback, помещается в 24GB VRAM

**Для эмбеддингов (бесплатно):**

```yaml
embedding:
  provider: local
  model: nomic-embed-text
  base_url: http://localhost:11434
```

## Переключение на лету

```
/model cerebras/llama-3.3-70b    # Полный путь модели
/model fast                       # Алиас
/model                            # Показать текущую модель
```

## Вспомогательные Модели (Модели Для Конкретных Задач)

Hermes поддерживает выделенные модели для восьми типов задач. Каждая может иметь своего провайдера, модель, base_url, api_key и timeout.

| Тип задачи | Что делает | По умолчанию |
|-----------|-------------|---------|
| `vision` | Анализ изображений/видео, понимание скриншотов | auto |
| `web_extract` | Суммирование скачанных веб-страниц | auto |
| `compression` | Сжатие контекста (суммирование старых сообщений) | auto |
| `session_search` | Поиск в прошлых транскриптах разговоров | auto |
| `approval` | Решение об авто-approve вызовов инструментов | auto |
| `skills_hub` | Обнаружение и сопоставление навыков | auto |
| `mcp` | Роутинг MCP инструментов | auto |
| `flush_memories` | Консолидация и очистка памяти | auto |

Когда установлено в `"auto"` (по умолчанию), Hermes проходит цепочку разрешения провайдеров: OpenRouter → Nous Portal → Custom endpoint → и т.д.

**Настройте в `~/.hermes/config.yaml`:**

```yaml
auxiliary_models:
  # Используйте быструю дешёвую модель для сжатия — она просто суммирует
  compression:
    provider: cerebras
    model: llama-3.3-70b
    timeout: 30

  # Используйте мультимодальную модель для анализа изображений/видео
  vision:
    provider: openrouter
    model: google/gemini-3.1-flash
    timeout: 60

  # Используйте локальную модель для поиска сессий (бесплатно, частые вызовы)
  session_search:
    provider: local
    model: nemotron:latest
    base_url: http://localhost:11434/v1
    api_key: ollama

  # Всё остальное остаётся на auto
  web_extract: auto
  approval: auto
  skills_hub: auto
  mcp: auto
  flush_memories: auto
```

**Зачем беспокоиться:**
- **Compression** запускается на каждой долгой сессии. Использование дешёвой/быстрой модели экономит деньги без влияния на качество (суммированию не нужен Opus).
- **Vision/video** нуждается в мультимодальной модели. Если ваша основная модель не поддерживает медиа, установите эту на ту, которая поддерживает.
- **Session_search** вызывается часто. Локальная модель делает его бесплатным.
- **Approval** контролирует авто-выполнение. Быстрая модель здесь означает меньше латентности на каждом вызове инструмента.

## Цепочка Fallback

Настройте автоматический fallback, если основная модель падает:

```yaml
model_fallback:
  - provider: cerebras
    model: llama-3.3-70b
  - provider: openrouter
    model: anthropic/claude-sonnet-5
  - provider: local
    model: nemotron:latest
```

Hermes пробует каждую по порядку. Если Cerebras упала, падает на OpenRouter, затем на локальную.

---

*Не запирайте себя в одного провайдера. Лучшая модель — та, которая достаточно быстрая и достаточно дешёвая для текущей задачи.*