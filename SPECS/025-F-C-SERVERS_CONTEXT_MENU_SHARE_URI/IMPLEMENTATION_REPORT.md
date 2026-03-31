# IMPLEMENTATION REPORT — 025-F-C-SERVERS_CONTEXT_MENU_SHARE_URI

- **Статус:** Completed  
- **Дата:** 2026-03-21  

## 1. Краткое резюме

На вкладке **Servers** добавлено **контекстное меню (ПКМ)** по строке списка прокси: первая строка — **тип из Clash API** (`type` в нижнем регистре или локализованная заглушка), вторая — **«Копировать ссылку»**, которая строит **subscription-style share URI** из **`config.json`** (`outbounds[]`, при отсутствии outbound — **WireGuard** в `endpoints[]`). Реализовано обратное кодирование к парсеру подписок в пакете `subscription`, поиск по тегу в `core/config`, обёртка строки для ПКМ во Fyne.

## 2. Как сделано контекстное меню

### 2.1 Почему не стандартный `widget.List` только

Список строится через `widget.NewList(createItem, updateItem)`. ПКМ должен срабатывать на **всей строке** (фон + подпись + кнопки Ping/Switch + gutter). Встроенного «secondary на строке» у `List` нет, поэтому корневой визуальный элемент строки оборачивается в **`internal/fynewidget.NewSecondaryTapWrap`**.

### 2.2 `SecondaryTapWrap`

- Наследует `widget.BaseWidget`, содержит один дочерний `CanvasObject` (у нас — `container.Stack` с фоном и контентом строки).
- Реализует **`desktop.Mouseable`**: на `MouseDown` с `SecondaryTapped` вызывается **`OnSecondary(*fyne.PointEvent)`**.
- Позиция для `PopUpMenu` берётся из **`pe.AbsolutePosition`**.

### 2.3 Сборка меню (`ui/clash_api_tab.go`)

- **`serversProxyContextMenu(ac, status, win, proxy)`** возвращает `*fyne.Menu` из двух пунктов:
  1. **`fyne.NewMenuItem(proxy.ContextMenuTypeLine(locale.T("servers.menu_context_type_unknown")), nil)`** — без `Disabled`, **`Action: nil`**: в Fyne цвет текста остаётся **foreground** (не `Disabled`); на десктопе тап не вызывает `trigger()`, меню можно закрыть только кликом вне или выбором второго пункта.
  2. **`fyne.NewMenuItem(locale.T("servers.menu_copy_link"), func() { ... })`** — запускает **`serversRunCopyShareURIToClipboard`**.

- **`serversRunCopyShareURIToClipboard`**: в горутине `ShareProxyURIForOutboundTag(cfgPath, tag)`; на UI-потоке через **`fyne.Do`**: ошибки → `ShowError` / `ShowErrorText` для `subscription.ErrShareURINotSupported`; успех → `Clipboard().SetContent(line)`, обновление `status`.

- Перед показом меню: выделение строки в `List`, `SetSelectedIndex`, обновление статуса «выбран прокси».

### 2.4 Откуда берётся тип для первой строки

- В **`api.GetProxiesInGroup`** после разбора `group["all"]` для каждого `name` читается `proxiesMap[name]`; если это объект, из него **`type`** (string) пишется в **`ProxyInfo.ClashType`**.
- **`ProxyInfo.ContextMenuTypeLine`**: `TrimSpace` → если пусто, вернуть аргумент `unknownLabel`; иначе **`strings.ToLower`**.

## 3. Как сделана сборка share URI

### 3.1 Принцип

Источник — **текущий sing-box JSON** в `config.json`, а не повторный fetch подписок. Форматы URI согласованы с **`ParseNode`** и документацией **`docs/ParserConfig.md`** (раздел Share URI).

### 3.2 Пакет `core/config/subscription` (`share_uri_encode.go`)

- **`ShareURIFromOutbound(out map[string]interface{})`**: диспетчер по `type`; для **`wireguard`** делегирует в **`ShareURIFromWireGuardEndpoint`** (одинаковая форма объекта endpoint).
- Реализации по протоколам зеркалят query/transport/tls туда, где уже есть зеркала при парсинге (`node_parser_transport.go` и т.д.).
- **`ShareURIFromWireGuardEndpoint`**: читает `private_key`, `address`, `peers[0]` (address, port, public_key, allowed_ips, опции), `mtu`, `listen_port` на endpoint, имя, dns; собирает `wireguard://`. Несколько peers → **`ErrShareURINotSupported`**.
- **`wireGuardPeerMaps`**: поддерживает и **`[]map[string]interface{}`** (как в памяти после парсера), и **`[]interface{}`** (после `json.Unmarshal`).

### 3.3 Пакет `core/config` (`outbound_share.go`)

- **`loadConfigRootMap`**: `getConfigJSON` + `json.Unmarshal` в `map[string]interface{}`.
- **`findTaggedInRoot`**: перебор `root[arrayKey]` как `[]interface{}`, сравнение `tag` у объектов.
- **`GetOutboundMapByTag` / `GetEndpointMapByTag`**: обёртки для тестов и прямого использования.
- **`ShareProxyURIForOutboundTag`**: при **пустом tag** → ошибка `empty outbound tag`; иначе **один** `loadConfigRootMap`, поиск в **`outbounds`**, при ошибке, подходящей под **`shareURITryEndpointAfterOutboundError`** (подстроки `not found` / `outbounds not found`), поиск в **`endpoints`** и **`ShareURIFromWireGuardEndpoint`**.

## 4. Локализация

| Ключ | Назначение |
|------|------------|
| `servers.menu_copy_link` | Пункт копирования |
| `servers.menu_context_type_unknown` | Если API не прислал `type` |
| `servers.copy_link_resolving` | Статус при сборке |
| `servers.copy_link_done` | Успех |
| `servers.copy_link_not_supported` | `ErrShareURINotSupported` |

## 5. Тесты

- **`core/config/subscription/share_uri_encode_test.go`**: round-trip VLESS/Trojan/SS, SOCKS из map, WireGuard round-trip, multi-peer WG → ошибка.
- **`core/config/outbound_share_test.go`**: vless в минимальном конфиге; WG только в `endpoints`; `GetEndpointMapByTag`.
- **`api/proxyinfo_test.go`**: `ContextMenuTypeLine`.

## 6. Документация (ссылки)

- **`docs/ParserConfig.md`** — раздел Share URI, GUI Servers.
- **`docs/ARCHITECTURE.md`** — `outbound_share`, `share_uri_encode`, `clash_api_tab`, `SecondaryTapWrap`.
- **`docs/TEST_README.md`** — перечисление тестовых файлов.
- **`docs/release_notes/upcoming.md`** — пользовательские и технические пункты релиза.

## 7. Связанные настройки (не часть SPEC меню, но в том же релизном коммите)

В **`bin/settings.json`** могут быть поля **`ping_test_url`** и **`ping_test_all_concurrency`**; при старте **`main.go`** вызывает **`api.SetPingTestURL`** / **`SetPingTestAllConcurrency`**. К контекстному меню не относится, но упоминается в **RELEASE_NOTES** и **upcoming**.

## 8. Проверки

- [x] `go test ./api/... ./core/config/...` (в среде без полной сборки Fyne UI — по возможностям CI/разработчика).
- [x] Локали: парность ключей en/ru для добавленных строк (`internal/locale` тесты).

## 9. Ограничения

- Один URI на один тег; WireGuard с **несколькими** peers не кодируется.
- Селекторы и прочие типы без кодировщика дают **ошибку** при копировании, а не тихий no-op.
