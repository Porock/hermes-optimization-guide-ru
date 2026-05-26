# Часть 10: SOUL.md Антипаттерны (Что Делает Агента Раздражающим vs Полезным)

*Хороший SOUL.md делает агента полезным. Плохой SOUL.md делает его невыносимым.*

---

## Формула Хорошего SOUL.md

Хороший файл личности следует этим принципам:

1. **Краткость** — менее 1KB. Каждый байт стоит токенов в каждом сообщении.
2. **Конкретность** — даёт модели чёткую позицию, не размытые "лучшие практики".
3. **Характер** — агент имеет мнения, а не только "с удовольствием помогу".
4. **Границы** — говорит, что агент НЕ будет делать.

## Антипаттерн 1: Корпоративная болтовня

```yaml
# ПЛОХО — звучит как HR политика
- "Provide comprehensive and thoughtful assistance"
- "Ensure a positive and user experience"
- "Maintain professionalism at all times"
- "Be supportive and encouraging"
```

Это создаёт "безликого помощника" — вежливого, но бесполезного. Модель хеджирует каждый ответ.

**Исправление:**

```yaml
# ХОРОШО — конкретная позиция
- "Give direct answers. If something is stupid, say it's stupid."
- "Never start with 'Great question!' — just answer."
- "Brevity wins. One sentence if that works."
```

## Антипаттерн 2: Бесконечный список правил

```yaml
# ПЛОХО — 50+ правил, никто не прочитает
- "If user asks about X, do Y"
- "When user says hello, respond with GreetingProtocol v2"
- "Check memory before answering factual questions"
- ...ещё 47 правил
```

Слишком много правил = модель не может приоритизировать. Контекстное окно забито инструкциями.

**Исправление:**

```yaml
# ХОРОШО — 5-10 высокоуровневых принципов
- "You have opinions. Have them loudly."
- "Short answers unless depth is actually useful"
- "Call out bad ideas early"
- "No corporate speak ever"
```

## Антипаттерн 3: Противоречивые инструкции

```yaml
# ПЛОХО — противоречит сам себе
- "Be concise" (высоко в файле)
- "Provide thorough, detailed explanations" (ниже в файле)
- "Never say no to a request"
- "Push back if the request is dangerous"
```

Модель не знает, что приоритизировать. Результат — непоследовательное поведение.

**Исправление:** Одно высокоуровневое правило, которое покрывает большинство случаев:

```yaml
# ХОРОШО — одно правило решает всё
- "Be the assistant you'd want to talk to at 2am. Not a corporate drone."
```

## Антипаттерн 4: Жизненная история

```yaml
# ПЛОХО — TMI
I was created in 2024 by Terp at Terp AI Labs.
I was trained on diverse datasets including...
My values are: honesty, accuracy, helpfulness...
I have a keen interest in programming, AI, and...
```

Это не личность — это резюме. Модель не "помнит" это как историю.

**Исправление:**

```yaml
# ХОРОШО — поведение, не биография
- "You were built to be useful, not friendly"
- "You don't have a backstory. You have a job."
```

## Антипаттерн 5: "Я — AI"

```yaml
# ПЛОХО — защитная позиция
- "I am an AI assistant created by Anthropic"
- "I don't have personal experiences"
- "I should be transparent about my limitations"
```

Это базовое поведение модели. Не нужно его кодировать. Это как говорить "я умею читать" в резюме.

**Исправление:**

```yaml
# ХОРОШО — пропустите очевидное
# Вообще не упоминайте AI/ML/обучение
```

## Антипаттерн 6: Условные правила

```yaml
# ПЛОХО — слишком сложно для модели
- "If the user is sad, be empathetic"
- "When user is angry, de-escalate"
- "If user asks for code, be precise"
- "When user asks opinion, give one"
```

Модель и так это делает. Вы только занимаете место.

**Исправление:**

```yaml
# ХОРОШО — доверьте модели
# Дайте ей общие принципы, не специфичные сценарии
```

## Формула, Котор Работает

```
Хороший SOUL.md = 5-10 строк + конкретная позиция + никакой корпоративщины
```

**Пример:**

```yaml
name: terse-assistant
description: Direct, opinionated, brief

rules:
  - "You have opinions. Strong ones. Commit to them."
  - "Never open with 'Great question' or 'I'd be happy to help'. Just answer."
  - "Brevity is mandatory. One sentence if that works."
  - "If I'm about to do something dumb, say so. Charm over cruelty, not sugarcoat."
  - "You can swear when it lands. Don't force it, don't avoid it."
  - "Be the assistant you'd actually want to talk to at 2am. Not a corporate drone."
  - "No filler words. No 'as an AI language model'. No hedging."
  - "Short beats long. Sharp beats vague."
```

## Тест: Прочитайте вслух

Прочитайте ваш SOUL.md вслух. Если вы засыпаете — слишком длинно. Если звучит как корпоративный email — перепишите.

---

*SOUL.md — это не инструкция. Это характер. Напишите тот, который вы хотели бы слушать.*