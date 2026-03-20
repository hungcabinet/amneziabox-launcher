# TASKS — 023-F-C-SUBSCRIPTION_TRANSPORT_VLESS_TROJAN

- [x] Добавить `node_parser_transport.go`: `uriTransportFromQuery`, `vlessTLSFromNode`, `trojanTLSFromNode`, plaintext-порты для эвристики TLS
- [x] Обновить `buildOutbound` в `node_parser.go` для VLESS и Trojan; VMess grpc → `service_name`; HTTP vmess host как список
- [x] Обновить `outbound_generator.go`: общая сериализация transport, порядок после flow, минимальный `tls` при `enabled:false`
- [x] `fetchAndParseSource`: `MakeTagUnique`
- [x] Тесты subscription + `TestGenerateNodeJSON_VLESS_WSTransportNoTLS`
- [x] Сверка с документацией sing-box: http `host` как массив; убрать `mode` из httpupgrade; убрать недокументированный `authority` из JSON
- [x] `docs/release_notes/upcoming.md`
- [x] SPEC / PLAN / IMPLEMENTATION_REPORT / README SPECS
- [x] SPEC: таблицы полей всех транспортов и TLS по доке sing-box; `SUBSCRIPTION_PARAMS_REPORT.md` с примерами из abvpn / goida / igareck
- [x] Парсер: `queryGetFold`, нормализация `alpn` (многослойный decode), `fp` → lowercase, `allowinsecure`, `tcp/raw` + `headerType=http` → transport `http`, `packetEncoding` → `packet_encoding`
- [x] Тесты на строках из указанных подписок (`TestParseNode_VLESS_TransportAndTLS` доп. подкейсы)
- [x] Опционально: `live_subscriptions_test.go` (`//go:build live`) — все vless/trojan/vmess из четырёх URL разбираются без ошибки

## Дополнения (после первичного закрытия 023)

- [x] **WS `headers.Host` из `sni`:** если в URI нет `host`, для `type=ws` подставляется `sni` (VLESS/Trojan — `uriTransportFromQuery`; VMess — `buildOutbound`).
- [x] **VLESS REALITY, голый TCP, без `flow`:** при пустом `flow`, непустом `pbk` и отсутствии транспорта `ws`/`grpc`/`http`/`xhttp` → outbound `flow: xtls-rprx-vision` (не для gRPC REALITY).
- [x] **Порядок в `source_loader`:** `NormalizeProxyDisplay` на тег после префикса, **до** `MakeTagUnique` (коллизии по нормализованным тегам).
- [x] **`internal/textnorm`:** единая нормализация тегов/лейблов (UTF-8, `❯`/`»`/`›` → ` > `); парсер, Clash `DisplayName`, визард.
- [x] **Clash API (`api/clash.go`):** `PathEscape` имён в `/proxies/{name}/delay` и в пути `PUT /proxies/{group}`; `QueryEscape` для параметра `url` в delay; `json.Marshal` тела переключения; тест `api/clash_url_test.go`.
- [x] **UI Servers:** `ProxyInfo.DisplayName` / `DisplayOrName()` для списка и трея; пинг/switch по **`Name`** (сырой тег из API).
- [x] Документация: `docs/ParserConfig.md`, `docs/release_notes/upcoming.md`, `docs/ARCHITECTURE.md`, этот TASKS / SPEC / IMPLEMENTATION_REPORT.
