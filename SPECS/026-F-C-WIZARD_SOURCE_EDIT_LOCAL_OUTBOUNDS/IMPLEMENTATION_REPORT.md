# Отчёт: 026-F-C-WIZARD_SOURCE_EDIT_LOCAL_OUTBOUNDS

**Статус:** реализовано.

**Дата:** 2026-03-22

## Кратко

- **Модель:** `core/config/configtypes/types.go` — `ProxySource.ExcludeFromGlobal`, `ExposeGroupTagsToGlobal`; `ParsedNode.SourceIndex`, `UnsetSourceIndex`.
- **Генерация:** `core/config/outbound_generator.go` — пул глобальных нод через `FilterNodesExcludeFromGlobal`; `collectExposeTagCandidates`; зависимости pass 2 для expose; `GenerateSelectorWithFilteredAddOutbounds(..., forGlobal, exposeCandidates)`.
- **Фильтры:** `core/config/outbound_filter.go` — `FilterNodesExcludeFromGlobal`, `ExposeTagSyntheticNode`, `SelectorFiltersAcceptNode`, `PreviewGlobalSelectorNodes`.
- **Превью визарда:** `ui/wizard/business/preview_cache.go` — выставление `SourceIndex`; `ui/wizard/outbounds_configurator/edit_dialog.go` — превью глобального селектора с exclude.
- **Бизнес визарда:** `ui/wizard/business/source_local_wizard.go` — маркеры, ensure/remove, синхронизация expose, переименование тегов при смене префикса.
- **UI:** `ui/wizard/tabs/source_edit_window.go` — вкладки **Настройки** / **Просмотр** (секция локальных `proxies[i].outbounds` + ноды подписки; при ошибке загрузки нод локальная секция сохраняется) / **JSON** (read-only `proxies[i]`); `afterSync` обновляет Preview и JSON при смене настроек, если вкладка активна. `source_tab.go` — Edit.
- **Тесты:** `core/config/expose_exclude_test.go`.
- **Документация:** `docs/ParserConfig.md`, `docs/release_notes/upcoming.md`.
- **Дедуп expose:** при сборке списка — `seenTags`; повтор того же тега из `addOutbounds` и expose не дублируется.
- **Миграция:** не требуется (версия 4, новые поля optional + `omitempty`).

## Проверки

- `go test ./core/config/...` — OK.
- `go test ./ui/wizard/business/...` — OK.
- Полная сборка с Fyne/CGO на среде без OpenGL — не запускалась.
