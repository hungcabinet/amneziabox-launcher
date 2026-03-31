# План: поиск причины потери фокуса (Windows)

## Подход

1. **Корреляция по времени** — временное логирование (или единичная отладочная сборка) с меткой времени для:
   - входа в **`fyne.Do`** из фоновых горутин (точечно: `updateVersionInfo` / `updateVersionInfoAsync`, при необходимости — узкий wrapper);
   - **`UIService.UpdateUI`** → `SetSystemTrayIcon` / цепочка обновления меню трея;
   - **`RequestFocus` / `Show`** на окнах лаунчера (grep по репозиторию + лог в критических местах).
2. **Сужение** — отключение/заглушка **по очереди** (локально или за build tag): цикл `startAutoUpdate`; обновление трея при неизменных данных; сравнить с субъективным интервалом 10–30 с.
3. **Окружение** — один монитор vs два; другая машина; версия Fyne в `go.mod` и известные issues upstream (поиск по `fyne-io/fyne` + Windows + focus / systray).

## Затрагиваемые зоны (ориентир)

| Зона | Файлы / пакеты |
|------|----------------|
| Автообновление версии Core | `ui/core_dashboard_tab.go` (`startAutoUpdate`, `updateVersionInfo`) |
| Трей | `core/uiservice/ui_service.go` (`UpdateUI`), `main.go` (`updateTrayMenu`, `SetSystemTrayMenu`) |
| Фокус окон | `core/uiservice/ui_service.go` (`ShowMainWindowOrFocusWizard`), `ui/wizard/...`, `core/tray_menu.go` |
| Платформа Win | `internal/platform/platform_windows.go`, при необходимости точечные проверки HWND (только если даст однозначный сигнал) |

## Риски

- Избыточное логирование в горячих путях — только под флагом или временная ветка.
- Нельзя утверждать «исправлено» без подтверждения у репортера или воспроизведения.

## Исход документации

- После этапа поиска: **IMPLEMENTATION_REPORT.md** в этой папке; при значимом итоге — строка в **docs/release_notes/upcoming.md** только когда будет **пользовательски значимый** фикс (не за этап чистой диагностики).
