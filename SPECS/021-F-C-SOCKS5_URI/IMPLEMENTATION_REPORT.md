## IMPLEMENTATION REPORT — 021-F-C-SOCKS5_URI

- **Status:** Completed
- **Date completed:** 2026-03-17

### 1. Summary

Добавлена поддержка парсинга URI-схем `socks5://` и `socks://` в Source, Connections и в теле подписки. Результат — outbound типа `socks` в sing-box (server, server_port, опционально username/password).

### 2. Implemented changes

**Парсер (`core/config/subscription/node_parser.go`):**
- В `IsDirectLink()` добавлено распознавание префиксов `socks5://` и `socks://`.
- В `ParseNode()` добавлена ветка для схемы socks: default port 1080, разбор host/port, опциональный user:password из userinfo, fragment для тега. Валидация: hostname обязателен.
- Извлечение пароля из userinfo для схемы `socks` (аналогично SSH/Trojan).
- В `buildOutbound()` добавлена ветка для `node.Scheme == "socks"`: type `socks`, server, server_port; при наличии — username (из node.UUID) и password (из Query).
- Обновлён комментарий пакета: добавлены SOCKS5 и WireGuard в список протоколов.

**Тесты (`core/config/subscription/node_parser_test.go`):**
- В `TestIsDirectLink` добавлены кейсы: SOCKS5 link, SOCKS5 with tag, SOCKS short form.
- Добавлен `TestParseNode_SOCKS5`: с авторизацией и тегом, без авторизации, socks:// с тегом, порт по умолчанию 1080, невалидный URI (отсутствует hostname). Проверка outbound type `socks` и полей server/server_port.

**Документация (`docs/ParserConfig.md`):**
- В примеры `connections` добавлена строка `socks5://user:pass@proxy.example.com:1080#Office SOCKS5`.
- В список парсируемых схем и поддерживаемых протоколов добавлены socks5:// и socks:// (SOCKS5).
- В описание секции «Генерация JSON узлов» добавлен SOCKS5.
- В таблицу поля `connections` добавлены socks5:// и socks://.
- Добавлен подраздел «SOCKS5 (socks5:// или socks://)» с форматом и примерами.

### 3. Tests & Checks

- [x] `go test ./core/config/subscription/... -run TestIsDirectLink` — все кейсы проходят.
- [x] `go test ./core/config/subscription/... -run TestParseNode_SOCKS5` — все кейсы проходят.
- [x] Существующие протоколы не затронуты (SOCKS5 — только добавление веток).

### 4. Risks / Limitations

- Нет. SOCKS5 даёт обычный outbound, стейдж и конфиг не менялись.

### 5. Notes

- Задача закрыта; папку переименовать в `021-F-C-SOCKS5_URI` (C = completed).
