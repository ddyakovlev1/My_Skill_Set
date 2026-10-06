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
| `karpathy-guidelines` | Правила против типичных ошибок LLM в коде: без переусложнения, точечные правки | свой |
| `prompt-engineer` | Превращает задачу в подробный мастер-промт или в промт для генерации изображения; вызывается только через `/prompt-engineer`, `/промт` или `/prompt-engineer-imagen`, `/промт-и` (режим изображения) | свой |
| `frontend-design` | Выразительный визуальный дизайн UI: эстетика, типографика, цвет | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/frontend-design) (Apache 2.0) |
| `brainstorming` | Уточнение идеи и дизайна перед любой творческой работой | [obra/superpowers](https://github.com/obra/superpowers) (MIT) |
| `writing-plans` | Пошаговый план реализации по спецификации | superpowers |
| `executing-plans` | Выполнение плана в текущей сессии | superpowers |
| `subagent-driven-development` | Выполнение плана через субагентов с ревью между задачами | superpowers |
| `dispatching-parallel-agents` | Параллельный запуск агентов на независимые задачи | superpowers |
| `test-driven-development` | TDD: сначала тест, потом код | superpowers |
| `systematic-debugging` | Поиск корневой причины бага до исправления | superpowers |
| `verification-before-completion` | Проверка командами перед заявлением «готово» | superpowers |
| `requesting-code-review` | Запрос ревью перед мержем | superpowers |
| `receiving-code-review` | Критичный разбор полученного ревью | superpowers |
| `using-git-worktrees` | Изоляция работы в git worktree | superpowers |
| `finishing-a-development-branch` | Завершение ветки: мерж, PR или очистка | superpowers |
| `writing-skills` | Создание и проверка новых скилов | superpowers |
| `using-superpowers` | Как находить и применять скилы (мета-скил) | superpowers |
| `diagnosing-superpowers` | Разбор, почему сессия со скилами пошла не так | superpowers |

## Заметки

- Скилы superpowers скопированы из коммита `8ca22db`. Ссылки вида `superpowers:<скил>` заменены на `<скил>`, потому что здесь скилы подключены без плагинного неймспейса.
- `frontend-design` взят из коммита `8a1541c` репозитория anthropics/skills.
- Лицензии чужих скилов лежат в их папках.
