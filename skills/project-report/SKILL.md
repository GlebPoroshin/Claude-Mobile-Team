---
name: project-report
description: Генерация финального отчёта о проделанной работе. Создаётся в конце пайплайна, после code review. Содержит summary, changes, commits, decisions, quality.
globs: []
autoContext: false
---

# Project Report Skill

Генерирует финальный отчёт о проделанной работе для пользователя.

## Когда использовать

После завершения всех этапов разработки и прохождения code review.

## Формат отчёта

```markdown
# Report: {Feature Name} [{TICKET}]

## Summary
Краткое описание что было сделано (2-3 предложения).

## Changes

### Shared ({N} files)
- Список изменённых/созданных файлов в shared модуле с кратким описанием

### Android ({N} files)
- Список изменённых/созданных файлов Android с кратким описанием

### iOS ({N} files)
- Список изменённых/созданных файлов iOS с кратким описанием (если применимо)

### Tests ({N} files)
- Список тестов

## Commits
- `[TICKET] commit message 1`
- `[TICKET] commit message 2`
- ...

## Architectural Decisions
Ключевые решения, принятые в процессе разработки.
- Решение 1: описание и обоснование
- Решение 2: описание и обоснование

## Quality
- detekt: {N} issues (0 = OK)
- ktlint: {N} issues (0 = OK)
- Tests: {N} passed, {M} failed
- Architecture compliance: PASS/FAIL
- Code review: APPROVED / CHANGES REQUESTED (and fixed)

## Notes
Дополнительные заметки, если есть (edge cases, known limitations, follow-up tasks).
```

## Правила

- Отчёт должен быть кратким и конкретным
- Не упоминать AI/Claude/агентов
- Фокус на результате, не на процессе
- Если были проблемы и фиксы — упомянуть кратко
- Если есть follow-up задачи — перечислить в Notes
