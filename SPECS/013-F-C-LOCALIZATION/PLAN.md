# План: локализация интерфейса (выбор языка)

## Архитектура

### Пакет `internal/locale`

Единый слой переводов, доступный из `core/` и `ui/`. Выбран `internal/locale` (не `ui/locale`), т.к. `core/tray_menu.go`, `core/error_handler.go` и другие файлы в `core/` содержат пользовательские строки — зависимость `core → ui` нарушила бы архитектурные инварианты.

### Формат хранения: JSON + go:embed

Переводы в файлах `en.json` / `ru.json` внутри пакета `internal/locale`. Встраиваются в бинарник через `go:embed`. Парсинг при старте (стандартный `encoding/json`, ~600 ключей — мгновенно). Без внешних зависимостей, без runtime-файлов.

Формат JSON-файлов — плоский объект `{ "key": "value" }`. Многострочные тексты — через `\n` в значениях. Пример:

```json
{
  "core.start": "Start",
  "core.stop": "Stop",
  "dialog.tun_help": "Enabling TUN will require entering your password...\n\nThis is because TUN mode needs elevated privileges."
}
```

### API пакета locale

```go
func T(key string) string             // перевод по ключу
func Tf(key string, a ...any) string  // перевод с fmt.Sprintf
func SetLang(lang string)             // сменить язык
func GetLang() string                 // текущий язык
func Languages() []string             // доступные языки
func LangDisplayName(code string) string // "en" → "English", "ru" → "Русский"
```

Fallback: текущий язык → English → сам ключ. Thread-safe (sync.RWMutex).

### Конвенция ключей

`{area}.{element}` — плоская, осмысленная:
- `core.start`, `core.stop`, `core.exit`
- `core.status_running`, `core.status_stopped`
- `wizard.tab_sources`, `wizard.tab_rules`
- `wizard.rules.add_rule`, `wizard.rules.final_outbound`
- `dialog.error.title`, `dialog.confirm.yes`
- `tray.start_vpn`, `tray.stop_vpn`, `tray.quit`
- `help.version`, `help.update_available`

### settings.json

Настройки лаунчера хранятся в `bin/settings.json` (путь: `GetBinDir(ExecDir)/settings.json`). Формат:

```json
{
  "lang": "en"
}
```

Расширяемая структура. Чтение при старте (синхронно, файл мал). Запись при смене языка. `"en"` по умолчанию.

Функции `LoadSettings(binDir) Settings` и `SaveSettings(binDir, Settings) error` — в `internal/locale/settings.go`.

### Runtime switching: частичный

Смена языка применяется:

**Немедленно (без доп. кода):**
- Wizard — создаётся заново при каждом открытии
- Tray menu — CreateTrayMenu() вызывается при каждом показе
- Все новые диалоги — создаются на лету
- Error messages — генерируются в момент вызова

**После рестарта приложения:**
- Главное окно: tab labels, содержимое табов Core/Servers/Diagnostics/Help

UX: при смене языка в Help tab показываем сообщение "Language changed to [X]. Restart the app to apply fully." / "Язык изменён на [X]. Перезапустите приложение для полного применения."

### Валидация (тест)

`locale_test.go` проверяет:
- Все ключи из en.json присутствуют в ru.json и наоборот
- Нет пустых значений
- Количество `%s`/`%d`/`%v` плейсхолдеров совпадает между языками

## Компоненты и изменения

### Фаза 1: Foundation

1. **internal/locale/locale.go** — ядро: T(), Tf(), SetLang(), GetLang(), Languages(), LangDisplayName(), init() с парсингом embed-файлов
2. **internal/locale/settings.go** — Settings struct, LoadSettings(), SaveSettings()
3. **internal/locale/en.json** — английские переводы (все ключи, заполняется инкрементально)
4. **internal/locale/ru.json** — русские переводы (все ключи, заполняется инкрементально)
5. **internal/locale/locale_test.go** — тест полноты переводов
6. **main.go** — загрузка settings.json при старте, вызов SetLang()
7. **ui/help_tab.go** — добавить Select виджет для выбора языка

### Фаза 2: Core tabs + Tray

8. **ui/app.go** — tab labels через T()
9. **ui/core_dashboard_tab.go** — все строки через T()/Tf()
10. **ui/help_tab.go** — все строки через T()/Tf()
11. **core/tray_menu.go** — все строки через T()
12. **core/error_handler.go** — ошибки через Tf()
13. **ui/diagnostics_tab.go** — все строки через T()/Tf()

### Фаза 3: Main tabs

14. **ui/clash_api_tab.go** — все строки через T()/Tf()
15. **ui/log_viewer_window.go** — все строки

### Фаза 4: Wizard

16. **ui/wizard/wizard.go** — заголовки, кнопки навигации
17. **ui/wizard/tabs/source_tab.go** — все строки
18. **ui/wizard/tabs/rules_tab.go** — все строки
19. **ui/wizard/tabs/preview_tab.go** — все строки
20. **ui/wizard/presentation/presenter_save.go** — сообщения сохранения
21. **ui/wizard/presentation/presenter_methods.go** — user-facing сообщения
22. **ui/wizard/presentation/presenter_async.go** — user-facing сообщения
23. **ui/wizard/presentation/presenter_sync.go** — user-facing сообщения

### Фаза 5: Dialogs + polish

24. **ui/wizard/dialogs/add_rule_dialog.go** — все строки
25. **ui/wizard/dialogs/save_state_dialog.go** — все строки
26. **ui/wizard/dialogs/load_state_dialog.go** — все строки
27. **ui/wizard/dialogs/get_free_dialog.go** — все строки
28. **internal/dialogs/dialogs.go** — все строки
29. **ui/wizard/outbounds_configurator/** — все строки
30. **core/process_service.go** — user-facing сообщения
31. **core/core_version.go** — user-facing сообщения
32. **ui/error_banner.go** — строки

### Фаза 6: Документация

33. **docs/release_notes/upcoming.md** — добавить раздел про локализацию
34. **docs/ARCHITECTURE.md** — описать пакет internal/locale

## Файлы (новые)

- `internal/locale/locale.go`
- `internal/locale/settings.go`
- `internal/locale/en.json`
- `internal/locale/ru.json`
- `internal/locale/locale_test.go`

## Файлы (изменяемые)

- `main.go`
- `ui/app.go`
- `ui/help_tab.go`
- `ui/core_dashboard_tab.go`
- `ui/clash_api_tab.go`
- `ui/diagnostics_tab.go`
- `ui/log_viewer_window.go`
- `ui/dialogs.go`
- `ui/error_banner.go`
- `core/tray_menu.go`
- `core/error_handler.go`
- `core/process_service.go`
- `core/core_version.go`
- `ui/wizard/wizard.go`
- `ui/wizard/tabs/source_tab.go`
- `ui/wizard/tabs/rules_tab.go`
- `ui/wizard/tabs/preview_tab.go`
- `ui/wizard/presentation/presenter_save.go`
- `ui/wizard/presentation/presenter_methods.go`
- `ui/wizard/presentation/presenter_async.go`
- `ui/wizard/presentation/presenter_sync.go`
- `ui/wizard/presentation/presenter_ui_updater.go`
- `ui/wizard/dialogs/add_rule_dialog.go`
- `ui/wizard/dialogs/save_state_dialog.go`
- `ui/wizard/dialogs/load_state_dialog.go`
- `ui/wizard/dialogs/get_free_dialog.go`
- `ui/wizard/outbounds_configurator/edit_dialog.go`
- `ui/wizard/outbounds_configurator/configurator.go`
- `internal/dialogs/dialogs.go`
- `docs/release_notes/upcoming.md`
- `docs/ARCHITECTURE.md`
