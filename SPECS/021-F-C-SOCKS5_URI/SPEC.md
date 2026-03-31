# SPEC: SOCKS5_URI — поддержка socks5:// в connections

Спецификация добавления парсинга URI-схемы `socks5://` в singbox-launcher: использование в Source, в Connections и в теле подписки (по одной ссылке на строку). Результат — outbound типа `socks` в sing-box.

---

## 1. Проблема

### 1.1 Текущее состояние

- В **Source** и **Connections** парсер поддерживает только ссылки вида `vless://`, `vmess://`, `trojan://`, `ss://`, `hysteria2://`, `hy2://`, `ssh://`, `wireguard://` (см. `IsDirectLink` и `ParseNode` в `core/config/subscription`).
- Пользователь не может добавить внешний SOCKS5-прокси (с логином/паролем или без) через вставку ссылки в Source/Connections, как в NekoRay и аналогах.
- sing-box поддерживает outbound `type: "socks"` с полями `server`, `server_port`, опционально `username`, `password`.

### 1.2 Что нужно

- Поддержать **формат URI `socks5://`** (и при необходимости `socks://` как синоним).
- Обрабатывать ссылки там же, где остальные прямые ссылки: **Source**, **Connections**, строки тела подписки по URL.
- Результат парсинга — одна нода (`ParsedNode`) с `Outbound` в формате sing-box outbound **type: "socks"**. SOCKS5-ноды попадают в массив **outbounds** (как vless/trojan/ss), без отдельной секции endpoints.

---

## 2. Требования

### 2.1 Распознавание и ветвление

- В `IsDirectLink()` добавить распознавание префиксов `socks5://` и `socks://` (после trim).
- В `ParseNode()` добавить ветку для схемы `socks5`/`socks`: разбор URI, извлечение host, port, опционально user/password из userinfo, fragment (#tag), построение `ParsedNode` с `Scheme: "socks"` и `Outbound` в виде sing-box outbound `type: "socks"`.

### 2.2 Формат URI

- **С авторизацией:** `socks5://user:password@host:port#tag`
- **Без авторизации:** `socks5://host:port#tag`
- Порт по умолчанию: **1080**, если не указан.
- Фрагмент `#label` — тег/комментарий ноды (как у остальных протоколов).
- Допустим синоним `socks://` с тем же форматом (нормализовать к схеме `socks` для sing-box).

**Примеры:**
```
socks5://myuser:mypass@proxy.example.com:1080#Office SOCKS5
socks5://proxy.example.com:1080
socks://127.0.0.1:1080#Local
```

### 2.3 Где действует

- **Source:** если значение — прямая ссылка `socks5://` или `socks://`, парсить как один узел.
- **Connections:** каждая строка в `ProxySource.Connections`, являющаяся такой ссылкой, парсится как один узел.
- **Подписка по URL:** строка, начинающаяся с `socks5://` или `socks://`, обрабатывается через `ParseNode` и даёт одну ноду.

### 2.4 Маппинг в sing-box outbound

Результат парсинга одной ссылки должен приводиться к структуре:

```json
{
  "type": "socks",
  "tag": "<tag ноды>",
  "server": "<host>",
  "server_port": <port>
}
```

При наличии user/password в URI добавить в outbound:
```json
"username": "<user>",
"password": "<password>"
```

- Тег ноды — из fragment (`#label`) или сгенерированный по правилам (например `socks-host-port`).
- Дедупликация тегов и префиксы/постфиксы источника (`tag_prefix`, `tag_postfix`, `tag_mask`) применяются так же, как для остальных протоколов.

### 2.5 Ошибки и валидация

- Обязательны: host и порт (или порт по умолчанию 1080). При отсутствии host — ошибка парсинга, нода не добавляется.
- Логирование через `debuglog` в точках start/success/error. Длина URI — в пределах существующего лимита (`MaxURILength`).

### 2.6 Критерии приёмки

- В Source или Connections можно вставить ссылку `socks5://user:pass@server.com:1080#Office SOCKS5` и после парсинга/обновления получить одну ноду с типом socks.
- В подписке (текст по URL) строка с одной ссылкой `socks5://...` парсится в одну ноду.
- Сгенерированный outbound совместим с sing-box: `type: "socks"`, `server`, `server_port`, при наличии — `username`, `password`.
- SOCKS5-ноды участвуют в селекторах и фильтрах наравне с остальными (входят в `allNodes`, фильтры по tag/scheme).
- Существующие протоколы (vless, vmess, trojan, ss, hysteria2, ssh, wireguard) работают без изменений.
- Документация `docs/ParserConfig.md` обновлена: в примерах connections и в списке поддерживаемых форматов указан `socks5://` (и при необходимости `socks://`).

---

## 3. Ограничения и не входит в задачу

- Не входит: поддержка SOCKS4 или иных вариантов SOCKS.
- Не входит: изменение стейджа генерации (endpoints, порядок секций) — SOCKS5 даёт обычный outbound.
- Изменения только в объёме парсера и `buildOutbound`; без смены версии ParserConfig.

---

## 4. Ссылки

- Парсер: `core/config/subscription/node_parser.go` (`IsDirectLink`, `ParseNode`, `buildOutbound`).
- Модель: `core/config/models.go` (`ParsedNode`).
- Документация форматов: `docs/ParserConfig.md`.
- Формат outbound socks в sing-box: [Configuration / Outbound / socks](https://sing-box.sagernet.org/configuration/outbound/socks/).
