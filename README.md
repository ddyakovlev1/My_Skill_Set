# My_Skill_Set

Личная коллекция скилов для Claude Code.

## Как подключить

Добавь этот репо в список репозиториев сессии Claude Code. Скилы лежат в `.claude/skills/`, и Claude Code подхватывает их оттуда автоматически.

Глобально на своей машине (необязательно):

```bash
git clone https://github.com/ddyakovlev1/My-Skill-Set ~/My-Skill-Set
mkdir -p ~/.claude/skills && cp -r ~/My-Skill-Set/.claude/skills/* ~/.claude/skills/
```

## Структура

Плоская: одна папка на скил, внутри `SKILL.md` и вспомогательные файлы.

```
.claude/skills/<имя-скила>/SKILL.md
```

## Как добавить скил

1. Создай папку `.claude/skills/<имя-скила>/`; имя папки должно совпадать с полем `name` в `SKILL.md`.
2. Положи туда `SKILL.md` (и лицензию, если скил чужой).
3. Добавь строку в таблицу ниже.

## Каталог

| Скил | Что делает | Источник |
|---|---|---|
| `prompt-engineer` | Превращает задачу в подробный мастер-промт для текстовой ИИ-модели; вызывается только через `/prompt-engineer` или `/промт` | свой |
| `prompt-engineer-imagen` | Превращает описание картинки в промт для генератора изображений (по умолчанию ChatGPT); вызывается только через `/prompt-engineer-imagen` или `/промт-и` | свой |
| `memory-backup` | Бэкап всех файлов памяти Claude в приватный GitHub-репо с автоматическим мержем в `main`; репо передаётся аргументом `owner/repo` | свой |
| `frontend-design` | Выразительный визуальный дизайн UI: эстетика, типографика, цвет | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/frontend-design) (Apache 2.0) |

## Архив

Неиспользуемые скилы лежат в папке `архив/`. Она вне `.claude/skills/`, поэтому Claude Code их **не подключает** по умолчанию.

Чтобы вернуть скил: перенеси его папку из `архив/` обратно в `.claude/skills/` и верни строку в каталог.

| Скил | Что делает | Источник |
|---|---|---|
| `karpathy-guidelines` | Правила против типичных ошибок LLM в коде: без переусложнения, точечные правки | свой |
| `brainstorming` | Уточнение идеи и дизайна перед любой творческой работой | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `writing-plans` | Пошаговый план реализации по спецификации | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `executing-plans` | Выполнение плана в текущей сессии | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `subagent-driven-development` | Выполнение плана через субагентов с ревью между задачами | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `dispatching-parallel-agents` | Параллельный запуск агентов на независимые задачи | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `test-driven-development` | TDD: сначала тест, потом код | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `systematic-debugging` | Поиск корневой причины бага до исправления | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `verification-before-completion` | Проверка командами перед заявлением «готово» | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `requesting-code-review` | Запрос ревью перед мержем | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `receiving-code-review` | Критичный разбор полученного ревью | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `using-git-worktrees` | Изоляция работы в git worktree | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `finishing-a-development-branch` | Завершение ветки: мерж, PR или очистка | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `writing-skills` | Создание и проверка новых скилов | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `using-superpowers` | Как находить и применять скилы (мета-скил) | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `diagnosing-superpowers` | Разбор, почему сессия со скилами пошла не так | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |

## Заметки

- Скилы superpowers скопированы из коммита `8ca22db`. Ссылки вида `superpowers:<скил>` заменены на `<скил>`, потому что здесь скилы подключены без плагинного неймспейса.
- `frontend-design` взят из коммита `8a1541c` репозитория anthropics/skills.
- Лицензии чужих скилов лежат в их папках.
