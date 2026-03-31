# План: редактор источника (Edit), `exclude_from_global`, `expose_group_tags_to_global`

## 1. Архитектура UI

- **Файл:** `ui/wizard/tabs/source_tab.go` (при росте — `ui/wizard/dialogs/`).
- **`showSourceEditWindow`:** заголовок, закрытие; **`AppTabs` / `DocTabs`**: **Настройки** | **Просмотр** (кэш превью / `RebuildPreviewCache` / `fetchAndParseSource` как сейчас).
- Префикс: редактирование в **Настройках**; в списке — только текст.
- Сериализация: как у `prefixEntry.OnChanged` (`SerializeParserConfig`, `InvalidatePreviewCache`, `ScheduleRefreshOutboundOptionsDebounced`).

## 2. Локальные auto/select и маркеры `WIZARD:`

- Бизнес-логика: `ui/wizard/business/` (или презентер) — идемпотентно, не трогать записи **без** маркеров в **`comment`**.
- Маркеры и теги — **SPEC.md**, разделы **«Новые поля»**, **§1**, **§2**, **§3**.

## 3. Два bool на `ProxySource`

Поведение полей — **SPEC.md**, раздел **«Новые поля `ProxySource`»** (таблица + синхронизация при снятии групп).

- **Код:** `core/config/configtypes/types.go` (+ алиас в `core/config` при необходимости).
- **`exclude_from_global`:** при **`true`** — ноды источника **не** участвуют в пуле кандидатов при генерации **каждого** элемента **`ParserConfig.outbounds`** (весь массив; тип записи **не** ограничивать — **та же ширина охвата**, что у **`expose_group_tags_to_global`**). **JSON** глобальных outbounds **не** редактировать из‑за этого флага.
- **`expose_group_tags_to_global`:** при **`true`** — на этапе **сборки** глобального outbound к **эффективному** списку подмешивать теги локальных групп источников с флагом (**SPEC §1–§2**; обход всего **`ParserConfig.outbounds`**). Если у записи заданы **`filters`** — каждый expose-кандидат прогонять через ту же логику, что ноды (**SPEC §5**, синтетика **`tag`/`comment`**, пустые **`host`** и др.); строки из JSON **`addOutbounds`** по-прежнему **без** **`filters`**. Сохранённый ParserConfig **не** меняется; дедуп/порядок — **IMPLEMENTATION_REPORT**. При **`false`** — не подмешивать теги этого источника.
- **Парсер:** `ParsedNode.SourceIndex`; для **всех** глобальных **`ParserConfig.outbounds`** — фильтрация пула / исключение нод при `ExcludeFromGlobal` (**`outbound_generator.go`**, `filterNodesForGlobalSelectors` и связанные пути — **PLAN §7**).
- **WireGuard:** исключение по `exclude_from_global` и для нод, уходящих в **`endpoints`**.

## 4. Миграция и версия

- По возможности **версия 4** + optional поля; иначе — `ParserConfigVersion`, `migrator.go`, **ParserConfig.md**.

## 5. Локализация

- **`internal/locale/en.json`**, **`ru.json`:** Edit, подвкладки, все чекбоксы из SPEC; tooltip для **«Теги в глобальных группах»** (нужна хотя бы одна локальная группа); текст предупреждения при **exclude** без пары auto+select.

---

## 6. Документация (обязательно при реализации)

| Документ | Что сделать |
|----------|-------------|
| **`docs/ParserConfig.md`** | Подраздел **`proxies[]` / ProxySource**: оба bool, генерация, без мутации **`outbounds[].addOutbounds`**. Кратко **SPEC §5**: expose-кандидаты и **`outbounds[].filters`**; **`addOutbounds`** из JSON не фильтруются. Маркеры **WIZARD:** — SPEC §2. |
| **`docs/release_notes/upcoming.md`** | После мержа фичи — EN/RU, кратко. |
| **`SPECS/026-F-C-…/IMPLEMENTATION_REPORT.md`** | Заполнено. |

Других файлов документации не трогать без необходимости; **`bin/wizard_template.json`** — только если появится требование в SPEC/отчёте.

## 7. Ориентир по файлам кода

| Зона | Файлы |
|------|--------|
| Модель | `core/config/configtypes/types.go`, при необходимости `ParsedNode` |
| Генератор | `core/config/outbound_generator.go`, при необходимости `outbound_filter.go` |
| Загрузка нод | проставить **`SourceIndex`** |
| Мигратор | `core/config/parser/migrator.go` |
| UI | `ui/wizard/tabs/source_tab.go`, `ui/wizard/business/*`, презентер при необходимости |
| Тесты | генератор, exclude + expose; expose + **`filters`** (в т.ч. отсев по **`host`**, проход по **`comment`** / **WIZARD:**) |

## 8. Риски

- Смена **`tag_prefix`** после создания локальных outbounds — синхронизация тегов в маркированных записях (**SPEC §1**).
- **`expose`:** эффективные списки длиннее JSON; дубликаты с **`addOutbounds`** — дедуп/порядок в отчёте. Пустые **`host`** у синтетики — expose не проходит AND с **`host`** (**SPEC §5**).
- Пустые глобальные outbound-списки при exclude без **`expose`** и без явных ссылок в JSON — UI-предупреждение.
