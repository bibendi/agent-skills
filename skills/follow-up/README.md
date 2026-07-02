# follow-up

Скилл для автоматической генерации follow-up заметок по транскрибациям встреч.

## Описание

Скилл анализирует транскрипцию встречи в формате Markdown и создает структурированный follow-up с:
- кратким summary встречи
- action items
- информацией об участниках

## Требования

- macOS (для launchd)
- [OpenCode](https://github.com/opencode-ai/opencode)
- [MacWhisper](https://apps.apple.com/us/app/whisper-transcription/id1668083311) — платное приложение (~5000 руб, единоразово)
- Homebrew-пакеты: `fswatch`, `jq`

## Установка

### 1. MacWhisper

1. Установить из App Store: [Whisper Transcription](https://apps.apple.com/us/app/whisper-transcription/id1668083311)
2. В настройках macOS включить доступ к микрофону: `Privacy & Security -> Microphone -> MacWhisper`
3. Рекомендуемая модель для русского языка: **Large V3 Turbo**

### 2. Зависимости

```bash
brew install fswatch jq
```

### 3. Скилл

```bash
npx skills add git@gitlab.sbmt.io:paas/ai/agent-skills.git --skill follow-up -a opencode
```

### 4. Watcher

```bash
./scripts/transcription-watcher setup
```

При установке можно переопределить модель:
```bash
OPENCODE_MODEL="opencode-go/gpt-4o" ./scripts/transcription-watcher setup
```

## Процесс записи

1. Во время встречи нажать `Record App Audio`, выбрать `Record All System Audio`, нажать `Record System Audio`
2. После встречи: `Stop Recording` → `Transcribe`
3. Разметить пользователей в формате "Имя Фамилия"
4. Экспортировать через `Export -> Transcript -> Format (.txt) -> Grouping (People)`
5. **Важно**: несмотря на выбор txt, в диалоге сохранения указать расширение `.md` — так файл корректно откроется в Obsidian
6. Сохранить в `Vault/Meetings/Transcriptions/` согласно конвенции именования
7. MacWhisper можно закрыть — дальше всё происходит автоматически

## Конвенции именования

Транскрипции сохраняются по шаблону:
```
Meetings/Transcriptions/{yyyy-mm-dd}-{meeting-type}.md
```

Примеры:
- `2026-02-28-team-daily.md`
- `2026-02-28-ivanov-1-to-1.md`
- `2026-03-01-all-hands.md`

Готовый follow-up появляется по аналогичному пути:
```
Meetings/Follow-Ups/{yyyy-mm-dd}-{meeting-type}.md
```

## Использование

### Автоматическая обработка

Watcher отслеживает папку `Meetings/Transcriptions/`. При появлении `.md` файла:
1. Запускается `opencode run` с промптом на обработку
2. Сессия автоматически удаляется (fire-and-forget)
3. Follow-up сохраняется в `Meetings/Follow-Ups/`
4. Лог операции пишется в `logs/`

### Ручной запуск

Если нужно обработать файл вручную или watcher не установлен:
```bash
opencode run \
  --dir "$VAULT_DIR" \
  --model "opencode-go/deepseek-v4-flash" \
  --format json \
  "Use 'follow-up' skill and process file 'Meetings/Transcriptions/2026-06-22-ps-daily.md'" \
  | jq -r '.sessionID // empty' \
  | head -n 1 \
  | xargs -I {} opencode session delete {}
```

## Управление watcher

```bash
# Статус
./scripts/transcription-watcher status

# Логи
./scripts/transcription-watcher logs

# Перезапуск
./scripts/transcription-watcher restart

# Остановка
./scripts/transcription-watcher stop

# Удаление сервиса
./scripts/transcription-watcher uninstall
```

## Листинг встреч в Obsidian

Для удобного просмотра и фильтрации follow-up можно настроить [Obsidian Base](https://obsidian.md/help/bases).

Шаблон конфигурации лежит в `templates/Follow-Ups.base`. Для подключения:

```bash
cp templates/Follow-Ups.base ../../Bases/Follow-Ups.base
```

Или создать файл `Bases/Follow-Ups.base` вручную и скопировать содержимое из `templates/Follow-Ups.base`.

## Синхронизация в Confluence

Для публикации follow-up в Confluence можно использовать плагин [obsidian-sync-confluence](https://github.com/dzplus/obsidian-sync-confluence).

Что умеет:
- Frontmatter-driven binding — привязка через `confluence_url`
- Автосоздание дочерних страниц через `confluence_parent_url`
- Поддержка Cloud и Self-hosted Confluence
- Загрузка локальных вложений в Confluence
- Mermaid / PlantUML pre-render

## Структура

```
.agents/skills/follow-up/
├── README.md              # эта инструкция
├── SKILL.md               # описание скилла для агента
├── scripts/
│   └── transcription-watcher    # скрипт автоматической обработки
├── templates/
│   └── Follow-Ups.base    # шаблон Obsidian Base для листинга встреч
└── logs/
    ├── .gitignore          # логи не попадают в git
    └── *.log               # файлы логов обработки
```
