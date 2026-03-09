# Задачи: Рефакторинг подсистемы Custom Rule (типы правил и вкладка Raw)

## Этап 1: Константы типов и модель state

- [x] Ввести константы типов правил (ips, urls, processes, srs, raw) в одном месте (rule_dialog.go или пакет моделей); заменить старые строки в rule_dialog.go
- [x] В wizard_state_file.go: PersistedCustomRule — добавить опциональное поле **Params**; для типа srs добавить опциональное поле **rule_set** (массив определений в формате bin/wizard_template.json: tag, type, format, url)
- [x] DetermineRuleType: возвращать только ips, urls, processes, srs, raw; логика «ровно одна группа полей» → иначе raw; убрать возврат "System"
- [x] При загрузке state: при отсутствии или старом формате type выводить тип через DetermineRuleType(rule); при сохранении записывать только константы
- [x] ToPersistedCustomRule: записывать type по константам; для processes/urls — params из состояния UI; для srs — rule_set из Rule.RuleSets
- [x] ToRuleState: восстанавливать из Params состояние UI для processes/urls; из rule_set — Rule.RuleSets для типа srs

## Этап 2: Диалог Add/Edit Rule — вкладки Form и Raw

- [x] Добавить вкладки Form и Raw (порядок: Form, затем Raw); Raw — многострочное поле с JSON правила
- [x] Form → Raw: при переключении подставлять в Raw JSON, собранный из текущей формы
- [x] Raw → Form: парсить JSON и выставлять тип/поля формы; при неудачном парсе — показать сообщение (правило не удалось распознать, форму загрузить нельзя), оставить вкладку Raw и тип raw
- [x] При сохранении с активной вкладки Raw — брать правило из JSON, тип выставлять raw; валидация (объект, outbound/action)
- [x] При открытии диалога добавления — в Raw заготовка/пример; при редактировании — текущее правило в JSON

## Этап 3: Form — тип srs и params

- [x] На форме тип srs: выбор rule-set'ов по SPEC 014 (каталог runetfreedom + ручной ввод URL); сохранение/загрузка rule + rule_set в state
- [x] Для типа processes: при сохранении записывать в params match_by_path, path_mode; при загрузке восстанавливать переключатель Simple/Regex из params
- [x] Для типа urls: при сохранении записывать в params состояние галочки «Regex»; при загрузке восстанавливать из params или по наличию domain_regex в rule

## Этап 4: Документация

- [x] WIZARD_STATE_JSON_SCHEMA.md: описание type (ips, urls, processes, srs, raw), params (назначение, примеры processes/urls), для srs — rule_set
- [x] Создать docs/WIZARD_STATE.md: формат файла state.json, структура JSON (custom_rules, type, params, rule_set для srs), DetermineRuleType, миграции
- [x] В docs/ARCHITECTURE.md добавить раздел про код и поток загрузки state (файлы, кто читает, ToRuleState, миграции, поток: файл → модель) со ссылкой на docs/WIZARD_STATE.md

## Этап 5: Проверка и отчёт

- [x] Сборка и тесты: go build ./..., go test ./..., go vet ./...
- [x] Проверить обратную совместимость: загрузка state со старыми значениями type
- [x] Заполнить IMPLEMENTATION_REPORT.md
