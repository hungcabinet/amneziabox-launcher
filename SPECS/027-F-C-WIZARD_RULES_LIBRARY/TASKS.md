# Задачи: Rules — custom + библиотека (027)

## Этап 1: State, засев, миграция

- [x] Признак/версия «миграция library выполнена»; условие срабатывания миграции со старого формата.
- [x] Миграция: блок из шаблона по порядку + старые `custom_rules`; перенос enabled/outbound из `selectable_rule_states`; очистка selectable при сохранении; идемпотентность.
- [x] Первый засев без сохранённого state: в `custom_rules` только пресеты с **`"default": true`** в `selectable_rules`, порядок как в шаблоне.
- [x] При необходимости — версия **WizardState** и правки в `wizard_state_file.go`.

## Этап 2: Клон и merge

- [x] Функция глубокого копирования пресета → запись `custom_rules` (rule / rules / rule_sets, тип через **DetermineRuleType**, 018).
- [x] `MergeRouteSection` и вызовы: после миграции только `custom_rules`.
- [x] Убрать отдельный UI-блок selectable после миграции; согласовать restore/save selectable.

## Этап 3: UI

- [x] Кнопка **Add from library**; модалка: скролл, чекбоксы, Cancel / **Add selected**; описание в tooltip/подстрочнике.
- [x] **Empty state** при пустом `custom_rules` (SPEC R8).
- [x] Строки locale (EN + RU в `bin/locale`).

## Этап 4: Закрытие

- [x] Ручная проверка (чеклист) — выполнено при приёмке; задача закрыта **2026-03-24**.
  1. **Новый профиль / нет `state.json`:** открыть Rules — список совпадает с пресетами шаблона с `"default": true`, порядок как в `wizard_template.json`; Final и outbound на строках осмысленны; Save → в `state.json` есть `rules_library_merged`, нет `selectable_rule_states`.
  2. **Add from library:** отметить 1–2 пресета → **Add selected** — копии в конце; тот же пресет ещё раз — вторая копия; Preview/Save — в `route.rules` порядок совпадает со списком (сверху базовые rules шаблона, например hijack-dns).
  3. **Пустой список:** удалить все правила (если возможно) или временный state с `custom_rules: []` и `rules_library_merged: true` — empty state + «Open library» / Add Rule.
  4. **Старый state (v2):** бэкап `state.json` с заполненным `selectable_rule_states` и без `rules_library_merged` — после Load порядок: блок шаблона по `selectable_rules` + сохранённые enabled/outbound, затем старые `custom_rules`; повторное открытие **не** дублирует правила (файл перезаписан с merged).
  5. **SRS-пресет:** без скачанных `.srs` — правило выключено или кнопка srs; после скачивания — можно включить, генерация без ошибок.
  6. **Get free / импорт:** применить `get_free.json` с selectable — после Load маршрут не дублирует selectable+custom (как после обычной миграции).
- [x] `go build ./...`, `go test ./...`, `go vet ./...`.
- [x] docs, `docs/release_notes/upcoming.md`, **IMPLEMENTATION_REPORT.md**.
