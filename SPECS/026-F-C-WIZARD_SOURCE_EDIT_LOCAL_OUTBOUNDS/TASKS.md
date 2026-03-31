# Задачи: Edit-окно источника, локальные auto/select, два bool на ProxySource

## Этап 1: Модель и парсер

- [x] **`ProxySource`:** **`exclude_from_global`**, **`expose_group_tags_to_global`** — см. **SPEC.md** раздел **«Новые поля»**.
- [x] **`ParsedNode.SourceIndex`** (или эквивалент); выставлять на всех путях в **`GenerateOutboundsFromParserConfig`**.
- [x] Все глобальные **`ParserConfig.outbounds`**: фильтрация пула нод по **`exclude_from_global`** (тип не ограничивать); локальные — **`nodesBySource[i]`**.
- [x] Тесты: exclude; **`expose`** + эффективный список; **`outbounds[].filters`** отсекают expose при несовпадении синтетики (**SPEC §5**); JSON **`addOutbounds`** без фильтра; сериализованный **`addOutbounds`** не меняется из‑за **`expose`**; локальные urltest/selector источника работают.
- [x] При необходимости: **`migrator`**, версия ParserConfig.

## Этап 2: UI — Edit

- [x] **View → Edit**, локали.
- [x] Табы **Настройки** / **Просмотр** (локальные outbounds + ноды) / **JSON** (read-only `proxies[i]`); live-обновление Preview/JSON при смене настроек на активной вкладке.
- [x] Настройки по **SPEC** (таблица UI): префикс, auto, select, два bool, предупреждение exclude; **`expose`** всегда виден, без локальных групп — **Disabled** + tooltip; **`expose`/`exclude`** только в **`proxies[]`**, без мутации **`ParserConfig.outbounds`** в JSON (**PLAN §3**).
- [x] Префикс только в Edit; в списке — отображение.
- [x] Предупреждение exclude без пары auto+select — ключи локалей.

## Этап 3: Сериализация и маркеры `WIZARD:`

- [x] Синхронизация галочек ↔ **`proxies[i].outbounds`** (**SPEC §1–§2**).
- [x] Сохранение валидного ParserConfig JSON.

## Этап 4: Документация и закрытие

- [x] **`docs/ParserConfig.md`** — по чеклисту **PLAN §6** (подраздел proxies, оба поля, пример, сценарии).
- [x] **`docs/release_notes/upcoming.md`**.
- [x] **`IMPLEMENTATION_REPORT.md`**, папка **`026-F-C-…`**.

## Проверки

- [ ] `go vet ./...`, `go build ./...`, `go test ./...` (полный прогон — CONSTITUTION / Fyne+CGO или CI).
- [x] Выборочно: `go vet ./core/config/... ./ui/wizard/business/...`, `go test` по тем же пакетам — успешно.
- [ ] Ручная проверка: Edit, exclude, expose, превью outbounds.
