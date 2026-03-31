# TASKS — 025-F-C-SERVERS_CONTEXT_MENU_SHARE_URI

Чеклист реализации (все пункты выполнены).

## Этап 1: Данные API и модель

- [x] Расширить api.ProxyInfo полем ClashType; заполнять в GetProxiesInGroup из node.type
- [x] Добавить ProxyInfo.ContextMenuTypeLine(unknownLabel string) string

## Этап 2: Кодирование URI

- [x] Реализовать subscription.ShareURIFromOutbound для поддерживаемых типов + ветка wireguard
- [x] Реализовать subscription.ShareURIFromWireGuardEndpoint (один peer)
- [x] wireGuardPeerMaps: []map и []interface{}

## Этап 3: Поиск в config.json

- [x] loadConfigRootMap, findTaggedInRoot, GetOutboundMapByTag, GetEndpointMapByTag
- [x] ShareProxyURIForOutboundTag: один разбор корня, outbound затем endpoint

## Этап 4: UI

- [x] internal/fynewidget/secondary_tap_wrap.go
- [x] clash_api_tab: serversProxyContextMenu, serversRunCopyShareURIToClipboard
- [x] Локали en/ru

## Этап 5: Тесты и документация

- [x] share_uri_encode_test.go, outbound_share_test.go, api/proxyinfo_test.go
- [x] ParserConfig.md, ARCHITECTURE.md, TEST_README.md, README.md, upcoming.md
