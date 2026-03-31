# PLAN: Servers — контекстное меню и share URI

## 1. Архитектура потока

```
Clash GET /proxies
    → GetProxiesInGroup: для каждого имени из group["all"] читать proxiesMap[name]["type"] → ProxyInfo.ClashType

UI widget.List (clash_api_tab)
    → createItem: строка = Stack(Rect, Padded(HBox(...))) обёрнут в SecondaryTapWrap
    → updateItem: OnSecondary → select row, serversProxyContextMenu(...)
        → fyne.Menu: [ ContextMenuTypeLine | nil ], [ Copy link → goroutine ShareProxyURIForOutboundTag ]

config.ShareProxyURIForOutboundTag(path, tag)
    → loadConfigRootMap (один json.Unmarshal корня)
    → findTaggedInRoot(..., "outbounds") → ShareURIFromOutbound
    → при ошибке «try endpoint»: findTaggedInRoot(..., "endpoints") → ShareURIFromWireGuardEndpoint

subscription.ShareURIFromOutbound / ShareURIFromWireGuardEndpoint
    → строка URI (согласованно с ParseNode / parseWireGuardURI)
```

## 2. Новые и изменённые модули

| Область | Файл | Назначение |
|--------|------|------------|
| API | `api/clash.go` | Поле `ProxyInfo.ClashType`, метод `ContextMenuTypeLine` |
| API | `api/proxyinfo_test.go` | Тест `ContextMenuTypeLine` |
| UI | `internal/fynewidget/secondary_tap_wrap.go` | Обёртка: secondary tap на верхний объект |
| UI | `ui/clash_api_tab.go` | `serversProxyContextMenu`, `serversRunCopyShareURIToClipboard`, обёртка строки |
| Config | `core/config/outbound_share.go` | `loadConfigRootMap`, `findTaggedInRoot`, `GetOutboundMapByTag`, `GetEndpointMapByTag`, `ShareProxyURIForOutboundTag` |
| Config | `core/config/outbound_share_test.go` | Тесты поиска и WG-only конфига |
| Subscription | `core/config/subscription/share_uri_encode.go` | Кодировщики URI + WireGuard endpoint |
| Subscription | `core/config/subscription/share_uri_encode_test.go` | Round-trip и edge cases |
| Локали | `internal/locale/en.json`, `bin/locale/ru.json` | Ключи меню и статусов |

## 3. Зависимости

- `getConfigJSON` / очистка JSONC — существующий код пакета `core/config`.
- `ShowError` / `ShowErrorText` — `ui/clash_api_tab.go`.
- `subscription.ErrShareURINotSupported` — сравнение через `errors.Is`.

## 4. Риски

- **ПКМ по кнопкам** внутри строки может не попадать в `SecondaryTapWrap` из-за иерархии hit-test Fyne — задокументировано в ParserConfig.
- Пустой **`type`** в API: показывается локализованная строка «тип неизвестен».
