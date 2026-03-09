# План: Рефакторинг подсистемы Custom Rule (типы правил и вкладка Raw)

## 1. Архитектура

### 1.1 Константы типов и state

- **Единый источник констант** типов правил: `ips`, `urls`, `processes`, `srs`, `raw` (файл с константами, например `rule_dialog.go`).
- **PersistedCustomRule** в state.json: поле **type** — только эти константы; опционально **params** (состояние UI по типам); для типа **srs** — опционально **rule_set** (массив определений rule-set'ов в формате **bin/wizard_template.json**, см. selectable_rules с SRS, напр. «Russian blocked resources»: `[{ "tag", "type", "format", "url" }, ...]`).
- **DetermineRuleType(rule)** возвращает только эти пять значений; при загрузке старых state тип всегда выводится из rule (маппинга старых строк нет).

### 1.2 Диалог Add/Edit Rule

- Вкладки **Form** и **Raw** (порядок: Form, затем Raw).
- Form: выбор типа по константам (ips, urls, processes, srs, raw), поля по типу; для srs — по SPEC 014.
- Raw: многострочное поле JSON правила; при сохранении с Raw — тип `raw`.
- При переключении Raw → Form: парсим JSON и заполняем форму; **при неудаче** — показываем сообщение, что правило не удалось распознать и форму загрузить нельзя, оставляем вкладку Raw и тип raw.

### 1.3 Документация

- **docs/WIZARD_STATE.md** — формат файла state.json, структура JSON (custom_rules, type, params, rule_set для srs, миграции).
- **docs/ARCHITECTURE.md** — раздел про код и поток загрузки state, перекрёстные ссылки на docs/WIZARD_STATE.md.

---

## 2. Изменения по файлам

### 2.1 ui/wizard/dialogs/rule_dialog.go

- Заменить константы типов на: `RuleTypeIP = "ips"`, `RuleTypeDomain = "urls"`, `RuleTypeProcess = "processes"`, `RuleTypeSRS = "srs"`, `RuleTypeCustom = "raw"` (или вынести в общий пакет моделей — один источник истины). Удалить старые длинные строки.

### 2.2 ui/wizard/models/wizard_state_file.go

- **PersistedCustomRule**: поле **Type** (string) — только константы; опциональное **Params** (map[string]interface{} или struct); для типа srs опциональное **RuleSet** (или **RuleSets**) — массив определений rule-set'ов в формате wizard_template (tag, type, format, url). Имя поля в JSON: **rule_set** по аналогии с bin/wizard_template.json.
- **ToPersistedCustomRule**: записывать type по константам; для processes/urls — заполнять Params из состояния UI; для srs — записывать rule_set из Rule.RuleSets.
- **ToRuleState**: при загрузке восстанавливать тип через DetermineRuleType(rule), если type отсутствует или в старом формате; из Params восстанавливать состояние UI для processes/urls; при наличии rule_set восстанавливать Rule.RuleSets.
- **DetermineRuleType**: возвращать только `ips`, `urls`, `processes`, `srs`, `raw`; логика распознавания по одной группе полей; иначе `raw`. Убрать возврат "System".

### 2.3 ui/wizard/dialogs/add_rule_dialog.go

- Радио-группа типов по новым константам; при открытии на редактирование — тип из сохранённого или DetermineRuleType(rule).
- Реализовать вкладки Form и Raw; при переключении Form → Raw подставлять собранный JSON в поле Raw; при переключении Raw → Form парсить JSON и заполнять форму; при неудачном парсе — сообщение пользователю («Could not recognize rule, form cannot be loaded; staying on Raw»), оставить Raw, тип raw.
- При сохранении с Raw — тип `raw`; для processes/urls записывать в params состояние UI.
- Для типа srs — UI по SPEC 014; сохранение/загрузка rule + rule_set.

### 2.4 SPECS/002-F-C-WIZARD_STATE/WIZARD_STATE_JSON_SCHEMA.md

- Описание **custom_rules[].type** — значения ips, urls, processes, srs, raw; описание **params** (назначение, примеры для processes и urls); для типа srs — поле **rule_set** (массив определений, как в wizard_template.json).

### 2.5 docs/WIZARD_STATE.md (новый)

- Формат файла state.json: назначение, версия, структура (version, id, comment, created_at, updated_at, parser_config, config_params, selectable_rule_states, custom_rules). Элемент custom_rules: label, type (ips/urls/processes/srs/raw), enabled, selected_outbound, description, rule, default_outbound, has_outbound, params, для srs — rule_set. Вывод типа из rule (DetermineRuleType), миграции v1→v2.

### 2.6 docs/ARCHITECTURE.md

- Добавить раздел про загрузку и хранение state визарда: в каких файлах хранится state, кто читает, ToRuleState, миграции, поток данных (файл → модель). Ссылка на docs/WIZARD_STATE.md для формата и структуры JSON.

---

## 3. Обратная совместимость

- При загрузке state: если type отсутствует или в старом формате — всегда выводить тип через **DetermineRuleType(rule)** (маппинга старых строк на константы нет).
- При сохранении всегда записывать только константы (ips, urls, processes, srs, raw).

---

## 4. Доработка задачи 014 (RULE_TYPE_SRS_URL) после окончания 018

Задача **014** (тип правила SRS) выполняется **после** завершения задачи 018. К моменту начала 014 подсистема custom rule уже будет с константами типов (ips, urls, processes, srs, raw), полем rule_set в state и общей логикой DetermineRuleType. Поэтому **после окончания 018** нужно доработать задачу 014: обновить её SPEC, PLAN и TASKS под эти итоги 018, чтобы при реализации 014 не дублировать и не расходиться с 018.

Ниже — что именно привести в соответствие с 018 в документации и реализации 014.

### 4.1 Константа типа

- В state и в коде тип правила SRS — **`srs`** (lowercase). В 014 в PLAN указано `RuleTypeSRSURL = "SRS"` — заменить на константу **`srs`** (в rule_dialog.go после 018 уже будет `RuleTypeSRS = "srs"`). В 014 использовать её, не вводить отдельную строку "SRS".

### 4.2 Хранение в state (PersistedCustomRule)

- Имя поля в JSON state для определений rule-set'ов — **`rule_set`** (как в **bin/wizard_template.json**). В 014 в PLAN упоминаются «RuleSets» / «rule_sets» — в 014 при реализации использовать имя **rule_set** и формат массива `[{ "tag", "type", "format", "url" }, ...]`, уже закреплённый в 018. Сохранение/загрузка: type "srs", rule, rule_set.

### 4.3 DetermineRuleType

- Логика распознавания типа **srs** после 018: в rule есть **только** `rule_set` (массив тегов), без ip_cidr, domain*, process_name/process_path_regex. В 014 в PLAN — «массив из одного элемента»; после 018 допускается один или несколько тегов, критерий — отсутствие других групп полей. В 014 опираться на реализацию DetermineRuleType из 018, не дублировать свою ветку для "SRS".

### 4.4 Диалог

- Тип SRS в радио-группе Add/Edit Rule — со значением константы **srs**; подпись в UI — по усмотрению (например «SRS»). Редактирование: тип из сохранённого `type` или из DetermineRuleType(rule).

**Итог:** после закрытия 018 обновить SPEC/PLAN/TASKS задачи 014 под константу `srs`, поле `rule_set` и общую логику DetermineRuleType; при реализации 014 опираться на уже сделанную в 018 подсистему.

---

## 5. Зависимости

- Тип **srs** и UI по нему (каталог runetfreedom, ручной ввод URL) — по **SPECS/014-F-C-RULE_TYPE_SRS_URL**. Структура хранения rule_set в state — по формату **bin/wizard_template.json** (selectable_rules с rule_set и rule).
