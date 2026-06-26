---
name: eval-harness
description: Automatic verification gates for agent actions — lint, format, test, file checks. Part of the Factory Model harness.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [harness, verification, evals, factory-model, quality]
    category: verify
---

# Eval Harness — Автоматические проверки

Автоматические верификационные гейты после каждого action агента. Часть Factory Model — AI генерит, harness проверяет.

> **WHEN TO APPLY:** после ЛЮБОГО изменения кода (write_file, patch, terminal с кодом) — НЕ жди запроса пользователя. Eval harness должен срабатывать как рефлекс.

## Гейты по типам операций

### 1. Код-гейты (TypeScript/React проекты)

После любого изменения `.ts/.tsx/.js/.jsx` файла в проекте:

```
① Формат: npx biome check --write <changed-files> 2>&1
② Линт:  bun run check 2>&1               # если script "check" существует в package.json
③ Билд:  bun run build 2>&1               # ТОЛЬКО если коммит-гейт
④ Тесты: bun test 2>&1                    # если script "test" существует
```

**Правила:**
- ① и ② — ВСЕГДА после изменения кода
- ③ — только перед git commit
- ④ — если затронуты файлы с логикой (не только стили/конфиги)
- Если любая проверка падает — остановись, объясни ошибку, спроси подтверждение прежде чем фиксить
- Если `bun run check` нет в проекте — не ошибка, просто пропусти

### 2. Файл-гейты

После `write_file`:
- **Файл существует** — `ls -la <path>` или `search_files`
- **Не пустой** — если байт > 0
- **Валидный формат** — для `.md`: frontmatter присутствует, wikilinks корректны
- Для `.json`: `python -m json.tool <path> > /dev/null`
- Для `.yaml/.yml`: `python -c "import yaml; yaml.safe_load(open('<path>'))"`

### 3. Git-гейты

После любого git action:
- **Ветка не переключилась неожиданно:** `git branch --show-current` — сверь с ожидаемой
- **Нет pending изменений** (если не планировался коммит): `git diff --stat`
- **Если нужен коммит** — после коммита: `git log -1 --oneline` — убедись что коммит прошёл

### 4. Wiki-гейты

После записи в Wiki (`C:/MediaServer/Wiki/wiki/`):
- **Файл создан/обновлён:** проверь через `search_files`
- **index.md обновлён:** grep для имени новой страницы в index.md
- **log.md обновлён:** grep для даты/названия в log.md
- **Wikilinks валидны:** все `[[...]]` ссылки в новой странице указывают на существующие страницы (по возможности)
- **Фронтматтер:** title, created, updated, type, tags, sources — все поля присутствуют

### 5. Коммит-гейты (перед `git commit` / открытием PR)

В дополнение к код-гейтам:
```
① Полный формат: npx biome check --write . 2>&1 || npx biome format --write .
② Полный линт:  bun run check 2>&1
③ Билд:         bun run build 2>&1       # если есть
④ Тесты:        bun test 2>&1             # если есть
⑤ Сводка:       git diff --stat
```

## Приоритеты проверок

**КРИТИЧЕСКИЕ (остановись немедленно):**
- Код не компилируется/не запускается
- Тесты падают
- Git в неожиданном состоянии (wrong branch, detached HEAD)

**ВАЖНЫЕ (остановись, сообщи, жди решения):**
- Линтер находит ошибки (не warnings)
- Форматтер сообщает о неотформатированных файлах
- Файл не создался / пустой после write

**INFO (сообщи, не останавливай):**
- Линтер warnings
- Файл больше 200 строк (кандидат на split)
- Нет AGENTS.md в проекте

## LM-as-a-Judge (для недетерминированных задач)

Для задач где нельзя написать детерминированный тест (summary, переводы, анализ):
- **Self-review:** после генерации summary/анализа — перечитай и задай 3 вопроса:
  1. Все ли ключевые пункты покрыты?
  2. Есть ли фактические ошибки?
  3. Адекватен ли тон/стиль?
- **Если ответ > 500 слов** — предложи TL;DR в начале

## Related Skills

- [[harness-engineering]] — umbrella skill: full Factory Model setup (AGENTS.md, CI, pre-commit)
- [[requesting-code-review]] — pre-commit security review pipeline

## Питфоллы

- **Не делай ③ (build) и ④ (tests) если проект не настроен** — пропусти, это не ошибка
- **Не исправляй падающие тесты автоматически без явного разрешения** — сначала объясни что упало
- **Biome format ≠ Biome check** — format авто-фиксит, check только проверяет. Используй format для фикса, check для верификации
- **Biome 2.x config schema изменилась:** `files.ignore` и `linter.rules` — неизвестные ключи в 2.5+. Для игнора используй `vcs.useIgnoreFile: true`, для линтера достаточно `"linter": {"enabled": true}` с дефолтными recommended правилами. Проверяй актуальный $schema URL.
- **Wiki-проверки не блокирующие** — если index.md забыл обновить, просто обнови, не останавливай всё
- **Не запускай biome format на чужих/неизменённых файлах** — только на тех что затронул
- **На Windows: `npx @biomejs/biome format --write .`** (полное имя пакета) — если biome не установлен глобально
- **`bun run check` нестандартный script** — в package.json может называться `lint`, `typecheck` или отсутствовать. Проверь scripts перед запуском.
