# Схема файла state.json


## Общая структура

```json
{
  "version": 2,
  "comment": "Описание состояния (опционально)",
  "created_at": "2026-01-30T12:00:00Z",
  "updated_at": "2026-01-30T12:00:00Z",
  
  "parser_config": {
    "version": 4,
    "proxies": [...],
    "outbounds": [...],
    "parser": {...}
  },
  "config_params": [
    {
      "name": "route.final",
      "value": "proxy-out"
    },
    {
      "name": "experimental.clash_api.secret",
      "value": "generated-secret-token"
    }
  ],
  "selectable_rule_states": [...],
  "custom_rules": [...]
}

---

## Детальное описание полей

### 1. Метаданные состояния

#### `version` (int, обязательное)
- **Тип:** `integer`
- **Значение:** `2` (текущая версия)
- **Описание:** Версия формата файла состояния. Используется для миграций. При загрузке старого формата (version=1) автоматически мигрируется в новый формат.

#### `id` (string, опциональное)
- **Тип:** `string`
- **Обязательность:** 
  - Для `state.json` — **не требуется** (имя файла является идентификатором)
  - Для именованных состояний (`<id>.json`) — **обязательно** и должно совпадать с именем файла (без `.json`)
- **Валидация:** 
  - Только буквы (a-z, A-Z), цифры (0-9), дефис (-), подчёркивание (_)
  - Максимальная длина: 50 символов
  - Должно совпадать с именем файла (для именованных состояний)
- **Пример:** 
  - Для `state.json`: поле отсутствует или `null`
  - Для `my-config.json`: `"my-config"`
  - Для `work-vpn.json`: `"work-vpn"`

#### `comment` (string, опциональное)
- **Тип:** `string`
- **Описание:** Комментарий/описание состояния для пользователя. Может использоваться в UI для отображения дополнительной информации о состоянии.
- **Пример:** `"Конфигурация для работы"`, `"Домашняя сеть с блокировкой рекламы"`, `""` (пустая строка) или поле отсутствует

#### `created_at` (string, обязательное)
- **Тип:** `string` (RFC3339)
- **Описание:** Дата и время создания состояния в формате RFC3339 (UTC)
- **Пример:** `"2026-01-30T12:00:00Z"`

#### `updated_at` (string, обязательное)
- **Тип:** `string` (RFC3339)
- **Описание:** Дата и время последнего обновления состояния в формате RFC3339 (UTC)
- **Пример:** `"2026-01-30T15:30:00Z"`

---

### 2. ParserConfig данные

#### `parser_config` (object, обязательное)
- **Тип:** `object` (JSON объект)
- **Описание:** Полная конфигурация парсера в формате JSON объекта. Это единственный источник ParserConfig для всего приложения:
  - Используется визардом при работе с конфигурацией
  - Используется при нажатии кнопки "🔄 Update" в главном окне для обновления конфигурации из подписок и для регулярных обновлений
- **Важно:** ParserConfig хранится только в `state.json`. При загрузке состояния визард и функция обновления используют этот блок.
- **Структура:** JSON объект с содержимым напрямую (без обертки `ParserConfig`), согласовано с `wizard_template.json`:
  ```json
  {
    "version": 4,
    "proxies": [
      {
        "source": "https://example.com/sub.txt"
      }
    ],
    "outbounds": [],
    "parser": {
      "reload": "1h",
      "last_updated": "2026-01-30T12:00:00Z"
    }
  }
  ```

**Важно:** 
- Это единственный источник ParserConfig для визарда
- При загрузке состояния визард использует этот блок
- При сохранении финальной конфигурации этот блок не копируется в `config.json`, остается только тут

---

**Примечание:** URL источников прокси хранятся в `parser_config.proxies` (поля `source` и `connections`). При восстановлении состояния URL'ы извлекаются из `parser_config.proxies` обратно в текстовый формат для отображения в UI.



---

### 4. Параметры конфигурации

#### `config_params` (array, обязательное)
- **Тип:** `array` of `ConfigParam`
- **Описание:** Параметры конфигурации, используемые при генерации финального конфига. Каждый параметр имеет имя (путь в JSON) и значение.
- **Структура элемента:**
  ```json
  {
    "name": "route.final",
    "value": "proxy-out"
  }
  ```
- **Поля элемента:**
  - `name` (string, обязательное) — путь к параметру в конфигурации, используя точечную нотацию (например, `"route.final"`, `"experimental.clash_api.secret"`)
  - `value` (string, обязательное) — значение параметра (может быть пустой строкой `""` для использования значения по умолчанию из шаблона)

**Стандартные параметры:**
- `route.final` — финальный outbound по умолчанию для правил маршрутизации (используется, если правило не имеет явного outbound)
- `experimental.clash_api.secret` — секретный токен для Clash API. Если отсутствует при сохранении конфига, генерируется случайный, но затем сохраняется в `state.json` для последующего использования

**Пример:**
```json
[
  {
    "name": "route.final",
    "value": "proxy-out"
  },
  {
    "name": "experimental.clash_api.secret",
    "value": "my-secret-token-12345"
  }
]
```

**Важно:**
- Единый шаблон (`config_template.json`) используется при генерации конфига (содержит `parser_config`, `config`, `selectable_rules`, `params`)
- Параметры из `config_params` применяются к шаблону при генерации
- Если параметр отсутствует в `config_params`, используется значение по умолчанию из шаблона (если есть)
- Параметр `experimental.clash_api.secret` генерируется автоматически при первом сохранении, если отсутствует, но затем сохраняется в `state.json`

---

### 5. Состояния правил из шаблона

#### `selectable_rule_states` (array, обязательное)
- **Тип:** `array` of `PersistedSelectableRuleState`
- **Описание:** Упрощённые состояния правил маршрутизации из шаблона. Хранят **только выбор пользователя** — определение правила (rule, description, rule_set, platforms и т.д.) берётся из шаблона при загрузке.

**Структура элемента `PersistedSelectableRuleState`:**
```json
{
  "label": "Block ads",
  "enabled": true,
  "selected_outbound": "reject"
}
```

**Поля `PersistedSelectableRuleState`:**
- `label` (string, обязательное) — название правила (используется как ключ для маппинга с шаблоном)
- `enabled` (boolean, обязательное) — включено ли правило в финальной конфигурации
- `selected_outbound` (string, обязательное) — выбранный outbound для правила (может быть "reject", "drop", или имя outbound)

**Пример массива:**
```json
[
  {
    "label": "Block ads",
    "enabled": true,
    "selected_outbound": "reject"
  },
  {
    "label": "Route Russian domains directly",
    "enabled": true,
    "selected_outbound": "direct-out"
  },
  {
    "label": "Messengers via proxy",
    "enabled": false,
    "selected_outbound": "proxy-out"
  }
]
```

**Важно:**
- Определение правила (rule, description, rule_set, platforms) **не дублируется** в state.json — берётся из шаблона
- При загрузке состояния selectable_rule_states маппятся на правила шаблона по `label`
- Если в шаблоне появилось новое правило, которого нет в state.json — оно инициализируется с default-значениями из шаблона
- Если в state.json есть правило, которого больше нет в шаблоне — оно игнорируется
- **Тип правила** (System/IP/Domain/Process и т.д.) определяется из шаблона, не хранится в state.json
- В режиме "Настроить заново" правила инициализируются из шаблона с `enabled=default`, `selected_outbound` из правила

---

### 6. Пользовательские правила

#### `custom_rules` (array, обязательное)
- **Тип:** `array` of `PersistedCustomRule`
- **Описание:** Пользовательские правила маршрутизации, созданные пользователем вручную (не из шаблона). Хранят полную структуру, т.к. не привязаны к шаблону.

**Структура элемента `PersistedCustomRule`:**
```json
{
  "label": "My custom rule",
  "type": "urls",
  "enabled": true,
  "selected_outbound": "proxy-out",
  "description": "Route specific domains to proxy",
  "rule": {
    "domain": ["custom.example.com"],
    "outbound": "proxy-out"
  },
  "default_outbound": "proxy-out",
  "has_outbound": true,
  "params": {}
}
```

**Поля `PersistedCustomRule`:**
- `label` (string, обязательное) — название правила
- `type` (string, опциональное) — тип правила; только константы: `ips`, `urls`, `processes`, `srs`, `raw`. При отсутствии или старом формате тип выводится из `rule` при загрузке (DetermineRuleType).
- `enabled` (boolean, обязательное) — включено ли правило
- `selected_outbound` (string, обязательное) — выбранный outbound
- `description` (string, опциональное) — описание правила
- `rule` (object, опциональное) — JSON-объект правила (domain, domain_suffix, ip_cidr, rule_set и т.д.)
- `default_outbound` (string, опциональное) — outbound по умолчанию
- `has_outbound` (boolean) — есть ли поле outbound в правиле
- `params` (object, опциональное) — состояние UI по типу правила; в конфиг не попадает. Для **processes:** `match_by_path` (bool), `path_mode` ("Simple"|"Regex"). Для **urls:** `domain_regex` (bool). Для типов `ips`, `srs`, `raw` может не использоваться.
- `rule_set` (array, опциональное) — только для типа `srs`: массив определений rule-set'ов в формате как в bin/wizard_template.json: `[{ "tag", "type", "format", "url" }, ...]`.

**Пример:**
```json
[
  {
    "label": "Work VPN domains",
    "type": "urls",
    "enabled": true,
    "selected_outbound": "proxy-out",
    "description": "Route work domains through VPN",
    "rule": {
      "domain": ["work.example.com", "internal.corp.com"],
      "outbound": "proxy-out"
    },
    "default_outbound": "proxy-out",
    "has_outbound": true
  },
  {
    "label": "Block trackers",
    "type": "urls",
    "enabled": true,
    "selected_outbound": "reject",
    "rule": {
      "domain_suffix": ["tracker.com", "analytics.net"],
      "action": "reject"
    },
    "has_outbound": false
  }
]
```

**Важно:**
- Эти правила создаются пользователем вручную
- Они не привязаны к шаблону, поэтому хранят полную структуру
- При восстановлении состояния они загружаются полностью (включая `rule`)

---


## Полный пример файла state.json

```json
{
  "version": 2,
  "comment": "Текущее рабочее состояние",
  "created_at": "2026-01-30T12:00:00Z",
  "updated_at": "2026-01-30T15:30:00Z",
  
  "parser_config": {
    "version": 4,
    "proxies": [
      {
        "source": "https://example.com/sub.txt"
      },
      {
        "source": "vless://uuid@server.com:443?encryption=none&security=tls&sni=server.com#Server1"
      }
    ],
    "outbounds": [],
    "parser": {
      "reload": "1h",
      "last_updated": "2026-01-30T12:00:00Z"
    }
  },
  
  "config_params": [
    {
      "name": "route.final",
      "value": "proxy-out"
    },
    {
      "name": "experimental.clash_api.secret",
      "value": "my-secret-token-12345"
    }
  ],
  
  "selectable_rule_states": [
    {
      "label": "Block ads",
      "enabled": true,
      "selected_outbound": "reject"
    },
    {
      "label": "Route Russian domains directly",
      "enabled": true,
      "selected_outbound": "direct-out"
    },
    {
      "label": "Messengers via proxy",
      "enabled": false,
      "selected_outbound": "proxy-out"
    }
  ],
  
  "custom_rules": [
    {
      "label": "Work domains",
      "type": "urls",
      "enabled": true,
      "selected_outbound": "proxy-out",
      "description": "Route work domains through VPN",
      "rule": {
        "domain": ["work.example.com"],
        "outbound": "proxy-out"
      },
      "default_outbound": "proxy-out",
      "has_outbound": true
    }
  ]
}
```

---

## Миграция со старого формата

### Формат v1 → v2

При загрузке `state.json` с `version: 1` (или без version) выполняется автоматическая миграция:

#### `selectable_rule_states`

**Старый формат (v1):**
```json
{
  "type": "System",
  "rule": {
    "label": "Block ads",
    "description": "Block advertisement domains",
    "raw": {
      "domain_suffix": ["ads.example.com"],
      "action": "reject"
    },
    "default_outbound": "reject",
    "has_outbound": false,
    "is_default": true
  },
  "enabled": true,
  "selected_outbound": "reject"
}
```

**Новый формат (v2):**
```json
{
  "label": "Block ads",
  "enabled": true,
  "selected_outbound": "reject"
}
```

**Логика миграции:** Из старого формата извлекается `rule.label`, `enabled` и `selected_outbound`. Остальное (description, raw, default_outbound, has_outbound, is_default, type) берётся из шаблона.

#### `custom_rules`

**Старый формат (v1):**
```json
{
  "type": "Domains/URLs",
  "rule": {
    "label": "Custom rule",
    "description": "Route specific domain",
    "raw": {
      "domain": ["custom.example.com"],
      "outbound": "proxy-out"
    },
    "default_outbound": "proxy-out",
    "has_outbound": true,
    "is_default": false
  },
  "enabled": true,
  "selected_outbound": "proxy-out"
}
```

**Новый формат (v2):**
```json
{
  "label": "Custom rule",
  "type": "Domains/URLs",
  "enabled": true,
  "selected_outbound": "proxy-out",
  "description": "Route specific domain",
  "rule": {
    "domain": ["custom.example.com"],
    "outbound": "proxy-out"
  },
  "default_outbound": "proxy-out",
  "has_outbound": true
}
```

**Логика миграции:** Из вложенного `rule` извлекаются поля на верхний уровень. `rule.raw` становится `rule`. `rule.label` становится `label`. `is_default` удаляется.

---

## Что НЕ сохраняется в state.json

### TemplateData (не сохраняется)
- **Причина:** Шаблон загружается из единого файла `config_template.json` при каждом запуске визарда
- **Содержимое:**
  - `ParserConfig` (string) — текст блока parser_config из шаблона. **Используется как fallback** при первой инициализации
  - `Config` (map[string]json.RawMessage) — секции конфига из шаблона (после применения params). **Всегда используются из шаблона** для генерации конфига
  - `ConfigOrder` ([]string) — порядок секций. **Всегда используется из шаблона**
  - `SelectableRules` ([]TemplateSelectableRule) — правила из шаблона (с label, description, rule, rule_set, platforms). **В режиме редактирования** маппятся с состояниями из state.json по label, **в режиме "Настроить заново"** используются для инициализации
  - `DefaultFinal` (string) — outbound по умолчанию из config.route.final. **Используется как fallback**, если не задан в `config_params`
- **Восстановление:** 
  - В режиме редактирования: шаблон загружается, правила маппятся по label с selectable_rule_states из state.json
  - В режиме "Настроить заново": шаблон используется полностью для инициализации нового состояния

### GeneratedOutbounds (не сохраняется)
- **Причина:** Генерируются из `parser_config_json` при парсинге
- **Восстановление:** При загрузке состояния запускается парсинг, и outbounds генерируются заново

### TemplatePreviewText (не сохраняется)
- **Причина:** Это кэш для preview, не критично для восстановления состояния
- **Восстановление:** Генерируется заново при необходимости

---

## Восстановление состояния

### Режим редактирования (по умолчанию)

При загрузке `state.json` (по умолчанию):

1. **Загрузить шаблон** из файла `config_template.json` — единый шаблон для всех платформ, с фильтрацией по `platforms` и применением `params` для текущей ОС
2. **Восстановить ParserConfig** из `parser_config`:
   - Используется визардом для работы с конфигурацией
   - Используется при нажатии кнопки "🔄 Update" в главном окне для обновления конфигурации
   - Это единственный источник ParserConfig
3. **Извлечь SourceURLs** из `parser_config.proxies` (поля `source` и `connections`) для отображения в UI

4. **Восстановить параметры конфигурации** из `config_params` (например, `route.final`, `experimental.clash_api.secret`)
5. **Восстановить состояния правил:**
   - Для `selectable_rule_states`: маппинг по `label` с правилами из шаблона. Применяются `enabled` и `selected_outbound` из state.json
   - Для `custom_rules`: загрузить полностью из файла (включая `rule`)
6. **Запустить парсинг** для генерации `GeneratedOutbounds`
7. **Синхронизировать GUI** с восстановленной моделью

**Важно:** В режиме редактирования selectable rules определяются шаблоном, а пользовательские выборы (enabled, selected_outbound) берутся из state.json. Custom rules загружаются полностью из state.json.

### Режим "Настроить заново"

При выборе опции "Настроить заново" в UI:

1. **Игнорировать `state.json`** — не загружать сохраненное состояние
2. **Загрузить шаблон** из файла `config_template.json`
3. **Инициализировать новое состояние** на основе шаблона:
   - Извлечь правила из шаблона (`selectable_rules`) в `selectable_rule_states` с дефолтными настройками из шаблона (`enabled = default из шаблона`)
   - Инициализировать `config_params`:
     - `route.final`: значение из `TemplateData.DefaultFinal` (если есть в шаблоне), иначе пустая строка `""`
     - `experimental.clash_api.secret`: **не инициализируется** — будет сгенерирован автоматически при первом сохранении конфига
   - Загрузить ParserConfig из шаблона (из секции `parser_config`)
4. **Запустить парсинг** для генерации `GeneratedOutbounds`
5. **Синхронизировать GUI** с новой моделью

**Важно:** Режим "Настроить заново" полезен, когда шаблон обновился и нужно начать с чистого листа, игнорируя старое сохраненное состояние.

---

## Валидация при загрузке

- `version` должен быть `1` или `2` (v1 мигрируется автоматически)
- `id` должен быть валидным (только разрешённые символы, макс. 50 символов)
- `parser_config` должен быть валидным JSON объектом с полями `version`, `proxies`, `outbounds`, `parser`

- `config_params` должен быть массивом
- Каждый элемент `config_params` должен иметь поля `name` и `value` (оба строки)
- `selectable_rule_states` должен быть массивом
- Каждый элемент `selectable_rule_states` должен иметь поля `label`, `enabled`, `selected_outbound`
- `custom_rules` должен быть массивом
- Каждый элемент `custom_rules` должен иметь поля `label`, `enabled`, `selected_outbound`, `rule`

---

## Размер файла

- **Максимальный размер:** 256 KB
- **Типичный размер:** 5-20 KB (упрощённый формат selectable_rule_states значительно уменьшил размер)
- При превышении лимита показывается предупреждение, но сохранение не блокируется
