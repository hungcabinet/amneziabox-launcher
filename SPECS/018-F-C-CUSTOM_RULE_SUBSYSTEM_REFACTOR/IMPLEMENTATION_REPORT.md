# Отчёт о реализации: Рефакторинг подсистемы Custom Rule (типы правил и вкладка Raw)

**Статус:** задача закрыта (018-F-C).

## Краткий план изменений

1. **Константы типов и модель state:** Введены константы `ips`, `urls`, `processes`, `srs`, `raw`. В `PersistedCustomRule` добавлены поля `params` (состояние UI) и `rule_set` (для типа srs). `DetermineRuleType` возвращает только эти пять значений; при загрузке при отсутствии/старом формате `type` тип выводится из `rule`. Реализованы запись/восстановление params и rule_set в `ToPersistedCustomRule` / `ToRuleState`.

2. **Диалог Add/Edit Rule:** Вкладки Form и Raw (порядок Form, затем Raw). Form → Raw: подстановка JSON из формы. Raw → Form: парсинг JSON и заполнение типа/полей; при неудачном распознавании — сообщение и остаёмся на Raw с типом raw. Сохранение с Raw — тип `raw`; валидация (объект, outbound/action). Добавлен тип SRS на форме (ручной ввод SRS URL, генерация tag и rule_set). Для processes/urls при сохранении записываются params (match_by_path, path_mode; domain_regex).

3. **Документация:** Обновлён `WIZARD_STATE_JSON_SCHEMA.md` (type, params, rule_set). Создан `docs/WIZARD_STATE.md` (формат state.json, custom_rules, DetermineRuleType, миграции). В `docs/ARCHITECTURE.md` добавлено описание хранения и загрузки state со ссылкой на `docs/WIZARD_STATE.md`.

## Изменённые файлы

- `ui/wizard/dialogs/rule_dialog.go` — подписи типов для UI и ключи (ProcessKey и т.д.); значения типов — в wizardmodels (единый источник истины)
- `ui/wizard/dialogs/add_rule_dialog.go` — вкладки Form/Raw, тип SRS, params при сохранении/загрузке, использование DetermineRuleType при редактировании
- `ui/wizard/models/wizard_state_file.go` — PersistedCustomRule (Params, RuleSet), DetermineRuleType (только 5 констант), ToPersistedCustomRule, ToRuleState
- `ui/wizard/template/loader.go` — поле Params в TemplateSelectableRule
- `SPECS/002-F-C-WIZARD_STATE/WIZARD_STATE_JSON_SCHEMA.md` — описание type, params, rule_set
- `docs/WIZARD_STATE.md` — новый файл
- `docs/ARCHITECTURE.md` — раздел про загрузку state
- `docs/release_notes/upcoming.md` — пункты по рефакторингу

## Команды для проверки

- `go build ./...` — в среде без OpenGL (GUI) может падать на драйвере Fyne; сборка пакетов `./ui/wizard/models/...` и `./ui/wizard/template/...` проходит.
- `go test ./...` — тесты не-GUI пакетов (по CONSTITUTION GUI-пакеты исключены из go test).
- `go vet ./ui/wizard/models/... ./ui/wizard/template/...` — проходит.

## Риски и ограничения

- Полный UI типа SRS по SPEC 014 (каталог runetfreedom Geosite/GeoIP, двухуровневый выбор) в этой задаче не реализован: сделан ручной ввод SRS URL и сохранение/загрузка rule_set. Задача 014 может дополнить каталог.
- Обратная совместимость: при загрузке старых state с полем `type` в старом формате тип выводится из `rule` (DetermineRuleType); маппинга старых строк на константы нет.

## Дополнительные доработки UI (по ходу задачи)

- **Rule name над вкладками:** Поле «Rule Name» вынесено над переключением Form/Raw; раскладка диалога — Border (сверху Rule name, снизу кнопки, центр — вкладки на всю высоту).
- **Domains/URLs — схема по центру:** Выпадающий список (Exact domains / Suffix / Keyword / Regex) справа от типа Domains/URLs, по центру строки — **показывается только при выбранном типе Domains/URLs** (аналогично галочке «Match by path» у Processes). Поддержка domain_suffix и domain_keyword на форме; при Raw→Form и при редактировании восстанавливаются режим и список.
- **Raw → Form: outbound:** При переключении с Raw на Form значение outbound в форме выставляется из поля `outbound` или `action` распознанного правила.
- **SRS — подсказка:** Кнопка «?» рядом с полем SRS URLs; по нажатию — информационное окно с рекомендацией искать rule-set'ы в runetfreedom, кнопка «Open» открывает ссылку в браузере.
- **Placeholder для Regex (Domains):** Для режима Regex в Domains/URLs задан поясняющий placeholder с примером регулярного выражения.
- **Кнопка Add при SRS:** Подключён OnChanged для поля SRS URLs, чтобы кнопка Add/Save разблокировалась при вводе URL без повторного изменения названия.

## Доработки после первого коммита

- **Единый источник констант типов:** Строковые значения типов (ips, urls, processes, srs, raw) заданы только в `wizardmodels`; в `rule_dialog.go` оставлены подписи для UI и ключи полей.
- **Params при сохранении:** В `ToPersistedCustomRule` выполняется поверхностная копия `Params`, чтобы последующая мутация `ruleState.Rule.Params` не влияла на сохранённые данные.
- **stripUTF8BOM:** Удаление BOM только в начале файла шаблона (убрана обработка в конце).
- **Дублирование:** Убран лишний вызов `pathModeRadio.SetSelected("Simple")`.

## Assumptions

- При переключении Raw → Form заполнение полей формы из распознанного JSON сделано для ips, urls, processes; для srs из rule нельзя восстановить список URL (в rule только теги), поэтому при смене вкладки на Form с Raw с правилом srs тип будет установлен, но поле SRS URLs остаётся пустым до ручного ввода или выбора из каталога (в задаче 014).
