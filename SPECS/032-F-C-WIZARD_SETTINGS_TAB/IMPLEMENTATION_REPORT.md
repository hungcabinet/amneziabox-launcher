# IMPLEMENTATION_REPORT: 032 WIZARD_SETTINGS_TAB

**Задача:** `SPECS/032-F-C-WIZARD_SETTINGS_TAB/` (статус **F-C**).  
**Связанные документы:** `SPEC.md`, `PLAN.md`, `TASKS.md` (все пункты выполнены).  
**Развёрнутый отчёт для чтения** (диаграммы, примеры JSON, решения): **`FINAL_READING_REPORT.md`**.

---

## Сделано (суть)

- Шаблон **`bin/wizard_template.json`:** **`vars`**, **`@…`**, TUN на macOS — **`darwin`** + **`if` / `if_or`**; **`darwin-tun`** не используется (в коде нет специальной обработки; метка в **`platforms`** не матчится с **`darwin`**).
- Пайплайн: **`ApplyTemplateWithVars`** → **`GetEffectiveConfig`** (учёт **`RawTemplate`**, **`SettingsVars`**, **`MaybeGenerateClashSecret`**); фильтр **`params`** по платформе и **`if`** / **`if_or`** (bool **`vars`**): если **`vars[].platforms`** не включает текущую ОС (**`VarAppliesOnGOOS`**), переменная для условия **ложна** до **`resolved`** / **`state.vars`** (**`ParamBoolVarTrue`**, **`vars_resolve.go`**; тесты в **`vars_resolve_test.go`**); подстановка **`@`**, числа для **`tun_mtu`** / **`mixed_listen_port`**.
- **`ValidateWizardTemplate`:** имена **`vars`**, уникальность, **`if`** только на **bool**, все **`@`** в **`config`** / **`params[].value`** объявлены; секция **`vars`** в JSON при валидации на **`@`** не сканируется.
- **State / LoadState:** **`WizardStateFile.Vars`**; в модель только имена из текущего шаблона; дубликаты **`name`** в JSON — побеждает последняя запись; миграция **`config_params.enable_tun_macos` → `vars.tun`** до **`restoreConfigParams`**.
- **UI:** вкладка **Settings** (перед **Preview**); **`wizard_ui`** hidden / view / edit; тип **`custom`** без строки; **Reset**; **`RefreshSettingsFromModel`** из **`SyncModelToGUI`**; TUN с **Rules** убран.
- **Исправление узкого места:** **`buildConfigSections`** и **`effectiveWizardConfig`** вызывают **`GetEffectiveConfig`**, если заполнен **`RawConfig`** и не пусты **`Params`** *или* **`Vars`** (раньше требовались только **`Params`** — ломалась подстановка при **`@`** только в **`config`**). Ошибка мержа — **`WarnLog`**, fallback на кэш **`TemplateData.Config`**.
- **Стабильность `clash_secret`:** **`MaterializeClashSecretIfNeeded`** (`business/create_config.go`) — пока значение «незаполнено» (**`ClashSecretUnresolved`**: пусто/пробелы или префикс **`CHANGE_THIS_`**), материализует результат **`MaybeGenerateClashSecret`** в **`SettingsVars`**; иначе при каждом **`GetEffectiveConfig`** превью/DNS могли бы видеть новый секрет только во временном **`resolved`**. Вызывается из **`BuildTemplateConfig`**, **`effectiveWizardConfig`**, **`LoadState`** (после **`restoreConfigParams`**), **`InitializeTemplateState`**. После **Сброс** по `clash_secret` ключ снова пуст — при следующем проходе генерируется новый.
- **Enum в UI:** если значение из state не входит в **`options`**, при сборке строки Settings подставляется первый вариант из шаблона и модель исправляется (**`MarkAsChanged`** только при фактическом изменении).
- **Платформенный `default_value`:** **`VarDefaultValue`** (**`vars_default.go`**) — объекты вида **`{"win7":"gvisor","default":"system"}`**; порядок ключей согласован с **`platforms`** (**`GOOS`**) + **`win7`** для **windows/386**; **`VarIndex`** / **`ResolveTemplateVars`** / **`SubstituteVarsInJSON`** пропускают только **`separator`**.
- **Разделитель Settings:** **`{"separator": true}`** в **`vars`** — линия между строками; не в state; **`presenter_state`** не включает в **`allowed`** пустое имя.
- **macOS, снятие `tun`:** пока **`RunningState.IsRunning()`** — нельзя выключить TUN в визарде (диалог **`wizard.settings.tun_off_core_running`**); после Stop — список целей: кеш из **`EffectiveConfigSection`("experimental")** + **`ExperimentalCacheFileFromSection`** (только внутри **`bin/`**), плюс **`logs/sing-box.log`** и **`logs/sing-box.log.old`** под **`ExecDir`** при **`Lstat`**; один привилегированный **`rm -rf`**; если логи удалены — **`FileService.ReopenChildLogFile()`** (**`settings_tun_darwin.go`**).

---

## Основные затронутые файлы

| Зона | Пути |
|------|------|
| Шаблон | `bin/wizard_template.json` |
| Загрузка / мерж / валидация | `ui/wizard/template/loader.go`, `vars_default.go`, `vars_resolve.go`, `substitute.go`, `template_validate.go` |
| Модель / state | `ui/wizard/models/wizard_model.go`, `wizard_state_file.go`, `wizard_settings_migrate.go` |
| Презентер | `ui/wizard/presentation/presenter_state.go`, `presenter_sync.go`, `gui_state.go` |
| UI | `ui/wizard/wizard.go`, `ui/wizard/tabs/settings_tab.go`, `settings_tun_darwin.go`, `settings_tun_stub.go`, `ui/wizard/tabs/rules_tab.go` |
| Сборка конфига / DNS | `ui/wizard/business/create_config.go`, `wizard_dns.go`, `materialize_clash_secret_test.go` |
| macOS TUN off / кеш | `core/config/config_loader.go` (**`ExperimentalCacheFileFromSection`**), `core/config/experimental_cache_test.go` |
| Локали | `internal/locale/en.json`, `bin/locale/ru.json` |
| Документация | `docs/WIZARD_STATE.md`, `docs/CREATE_WIZARD_TEMPLATE.md`, `docs/CREATE_WIZARD_TEMPLATE_RU.md`, `docs/ARCHITECTURE.md`, `docs/release_notes/upcoming.md`, `RELEASE_NOTES.md`, `SPECS/README.md`, `SPEC.md`, `PLAN.md` |
| Тесты | `ui/wizard/models/wizard_settings_migrate_test.go`, `ui/wizard/template/vars_default_test.go`, `vars_resolve_test.go`, `template_validate_test.go`, `substitute_test.go`, `apply_template_test.go` |

---

## Тесты и сборка

- `go build ./...`
- `go test ./...`
- `go vet ./...`
- Дополнительно проверялось: `./build/build_darwin.sh arm64` (без **`-i`** в `/Applications`).

---

## Ручной регресс (чеклист)

1. Вкладки: … → **Rules** → **Settings** → **Preview**; на macOS в Settings есть **tun** и связанные поля.
2. Изменение **log_level** / **clash_api** → превью / сохранение отражают **`vars`** в **state.json**.
3. macOS: **tun** off → в превью нет TUN-inbound; on → снова есть.
4. macOS: ядро **Running** → снять **tun** на Settings нельзя (диалог); после **Stop** — снятие и при необходимости запрос пароля на удаление кеша под **`bin/`** и **`logs/sing-box.log`** / **`.old`**.
5. Старый state с **`enable_tun_macos`** без **`vars.tun`** → после Load в модели есть **`tun`**.
6. **Reset** снимает override (до Save ключ может отсутствовать в файле).
7. Значение **clash_secret** не логируется при штатной работе (CONSTITUTION).

---

## Ограничения и риски

| Риск / ограничение | Как проверить |
|--------------------|---------------|
| Нет жёсткой валидации порта/CIDR в UI | Ввести мусор в **tun_mtu** / порт mixed — в логе warn от substitute, в JSON число 0 или пустая строка по правилам кода. |
| **enum** не из **`options`** в state | При сборке Settings подставляется первый вариант из шаблона; при ручной правке JSON до открытия визарда редкий случай уже обрабатывается в UI. |
| Сироты в **`model.SettingsVars`** (не из шаблона) | Не пишутся в state при Save; в памяти до перезагрузки не влияют на resolve. |
| Локали кроме **en**/**ru** | Новые ключи только в **en** + **ru**; остальные каталоги — fallback (см. `internal/locale`). |
| **macOS:** **`tun`** off при «зомби» sing-box вне лаунчера | Проверка только **`RunningState`**; если лаунчер считает **Stopped**, а процесс ещё жив, снятие **tun** разрешено — кеш может не удалиться без прав; пользователь видит ошибку при **`rm`** или останавливает процесс вручную. |

---

## Закрытие Spec Kit

- Папка **`032-F-C-WIZARD_SETTINGS_TAB`**, **SPEC.md** со статусом **C**, **SPECS/README.md** обновлён.
- Пользовательская выжимка: **`RELEASE_NOTES.md`** + **`docs/release_notes/upcoming.md`**.

---

## Соответствие контракту `SPECS/IMPLEMENTATION_PROMPT.md`

| Пункт контракта | В отчёте / артефактах |
|-----------------|------------------------|
| Краткий план изменений | Секция **«Сделано»** |
| Список файлов | Таблица **«Основные затронутые файлы»** |
| Ключевые фрагменты кода | Не дублируются в отчёте — см. функции **`ValidateWizardTemplate`**, **`restoreConfigParams`**, **`CreateSettingsTab`**, условие **`GetEffectiveConfig`** в **`create_config.go`** / **`wizard_dns.go`** |
| Команды проверки | **«Тесты и сборка»** |
| Риски и ограничения | Таблица выше |
| Предположения | Шаблон поставляется с **`vars`**, если в **`config`**/**`params`** есть **`@`**; иначе **`ValidateWizardTemplate`** при загрузке вернёт ошибку. Внешние профили **get_free** не задают **`vars`** — поведение как у state без overrides. |
