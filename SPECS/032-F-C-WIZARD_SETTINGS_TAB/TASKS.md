# TASKS: WIZARD_SETTINGS_TAB (032)

Чеклист по **PLAN.md** и **SPEC.md**. Решения продукта — **PLAN §13**.

## Этап A — Шаблон и парсинг

- [x] Обновить **`bin/wizard_template.json`**: **`vars`**, **`@…`**, **`"if": ["tun"]`** для TUN на **`darwin`**, **`if_or`**, без **`darwin-tun`**.
- [x] Задокументировать: при **`params.if` / `if_or`** переменная с **`vars[].platforms`**, не содержащим текущую ОС, даёт **false** (независимо от **`state.vars`**); SPEC, PLAN, **docs/CREATE_WIZARD_TEMPLATE** (EN/RU), **docs/ARCHITECTURE.md**, **docs/WIZARD_STATE.md**; юнит-тесты в **`vars_resolve_test.go`**.
- [x] Расширить десериализацию корня шаблона: **`Vars`**, **`TemplateParam.If`**.
- [x] Валидация: все **`@name`** объявлены; **`if`** только на **`bool`**-имена (PLAN §13); уникальность **`name`**; **`{"separator": true}`** без конфликта с полями переменной (**`template_validate.go`**).
- [x] **`default_value`**: тип **`VarDefaultValue`** (скаляр или объект **`GOOS`/`win7`/`default`**); **`tun_stack`** в шаблоне; **`vars_default.go`** / тесты.

## Этап B — Разрешение и подстановка

- [x] **`ResolveVars`**: state → default_value → default_node; **`text_list`** ↔ newline в **`value`**.
- [x] **`clash_secret`**: пусто или префикс **`CHANGE_THIS_`** → генерация (**`crypto/rand`**, 16 символов **`[A-Za-z0-9]`**); не логировать.
- [x] **`enableTunForDarwin`** из разрешённого **`tun`** (**`"true"`**).
- [x] **`applyParams`**: фильтр по **`matchesPlatform`**, затем по **`if`** (bool).
- [x] Подстановка **`@<name>`** по списку объявленных имён; числа для **`tun_mtu`** / **`mixed_listen_port`**; warn по SPEC при пустом итоге.

## Этап C — State и миграция

- [x] **`state.json`**: ключ **`"vars"`**, элементы **`{ "name", "value" }`**; сироты при Load.
- [x] Миграция **`config_params.enable_tun_macos`** → **`vars`** запись **`tun`**; убрать дублирующую логику из кода после миграции (PLAN §12 п. 2).
- [x] **docs/WIZARD_STATE.md**.

## Этап D — UI Settings

- [x] Вкладка **Settings** (предпоследняя перед Preview): **`wizard.go`** + **`settings_tab.go`** (или аналог).
- [x] Строки из **`TemplateData.Vars`**, **`wizard_ui`**, **`platforms`**; подписи **`title`** / **`tooltip`** (локали); разделители **`separator`** (**`settingsSeparatorBlock`** в **`settings_tab.go`**).
- [x] Контролы: text, bool, enum, **text_list** (многострочное, ~3 строки min).
- [x] Дефолт / Сброс / запись в модель и в сериализуемый **`vars`** по SPEC.

## Этап E — Удаление старого TUN UI

- [x] Убрать галочку TUN с **Rules**; источник истины — **`tun`** на Settings + миграция state.

## Этап F — Тесты и доки

- [x] Юнит-тесты: resolve, **`if`**, подстановка, миграция, **`clash_secret`** (подмена **`ClashSecretReader`**).
- [x] **docs/CREATE_WIZARD_TEMPLATE.md**, **docs/release_notes/upcoming.md**, при необходимости **ARCHITECTURE.md**.

## Этап G — Финализация (PLAN §12)

- [x] Сборка: **`go build` / `./build/build_darwin.sh arm64`** (см. **IMPLEMENTATION_REPORT**; **`-i`** — по желанию локально).
- [x] Слои, GUI-практики, без лишних обёрток; сценарии ручного теста; ограничения в **IMPLEMENTATION_REPORT**; финальный проход по коду.
