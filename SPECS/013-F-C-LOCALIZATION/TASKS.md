# Задачи: 013 — локализация интерфейса

## Фаза 1: Foundation

- [x] internal/locale/locale.go: T(), Tf(), SetLang(), GetLang(), Languages(), LangDisplayName(), init() с go:embed JSON
- [x] internal/locale/settings.go: Settings struct, LoadSettings(), SaveSettings() для bin/settings.json
- [x] internal/locale/en.json: английские переводы (все ключи)
- [x] internal/locale/ru.json: русские переводы (все ключи)
- [x] internal/locale/locale_test.go: тест полноты переводов (ключи, пустые значения, плейсхолдеры)
- [x] main.go: загрузка settings.json при старте, вызов locale.SetLang()
- [x] ui/help_tab.go: виджет Select для выбора языка, сохранение в settings.json

## Фаза 2: Core tabs + Tray

- [x] ui/app.go: tab labels через T()
- [x] ui/core_dashboard_tab.go: все user-facing строки через T()/Tf()
- [x] ui/help_tab.go: все user-facing строки через T()/Tf()
- [x] core/tray_menu.go: все строки через T()
- [x] core/error_handler.go: ошибки через Tf()
- [x] ui/diagnostics_tab.go: все строки через T()/Tf()

## Фаза 3: Main tabs

- [x] ui/clash_api_tab.go: все user-facing строки через T()/Tf()
- [x] ui/log_viewer_window.go: все строки

## Фаза 4: Wizard

- [x] ui/wizard/wizard.go: заголовки, кнопки навигации, диалоги
- [x] ui/wizard/tabs/source_tab.go: все строки
- [x] ui/wizard/tabs/rules_tab.go: все строки
- [x] ui/wizard/tabs/preview_tab.go: все строки
- [x] ui/wizard/presentation/presenter_save.go: сообщения сохранения
- [x] ui/wizard/presentation/presenter_methods.go: user-facing сообщения
- [x] ui/wizard/presentation/presenter_async.go: user-facing сообщения
- [x] ui/wizard/presentation/presenter_sync.go: user-facing сообщения
- [x] ui/wizard/presentation/presenter_ui_updater.go: user-facing сообщения

## Фаза 5: Dialogs + остальное

- [x] ui/wizard/dialogs/add_rule_dialog.go: все строки
- [x] ui/wizard/dialogs/save_state_dialog.go: все строки
- [x] ui/wizard/dialogs/load_state_dialog.go: все строки
- [x] ui/wizard/dialogs/get_free_dialog.go: все строки
- [x] internal/dialogs/dialogs.go: все строки
- [x] ui/wizard/outbounds_configurator/edit_dialog.go: все строки
- [x] ui/wizard/outbounds_configurator/configurator.go: все строки
- [x] ui/error_banner.go: строки
- [x] ui/dialogs.go: строки
- [x] core/process_service.go: user-facing сообщения
- [x] core/core_version.go: user-facing сообщения
- [x] core/controller.go: user-facing сообщения

## Фаза 6: Документация

- [x] docs/release_notes/upcoming.md: раздел про локализацию
- [x] docs/ARCHITECTURE.md: описать пакет internal/locale
