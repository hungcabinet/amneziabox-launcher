# SUBSPEC: Servers context menu — Copy server link / Copy jump server link

## 1. Контекст

После реализации `detour`-цепочек (main + jump) один share URI не описывает оба hop одновременно в стандартном формате.
На вкладке **Servers** сейчас пользователь видит только общий пункт копирования ссылки и получает `not supported` для outbound с `detour`.

## 2. Цель

Добавить понятный UX для копирования ссылок при цепочке:

- **Copy server link** — копирует URI текущего (основного) outbound.
- **Copy jump server link** — копирует URI outbound, на который ссылается `detour`.

Это не меняет стандарты URI и не вводит launcher-only схему.

## 3. Scope

### In scope

1. Контекстное меню строки proxy на вкладке **Servers**:
   - пункт `Copy server link` (вместо/вместе с текущим `Copy link`);
   - пункт `Copy jump server link` (активен только если у outbound есть непустой `detour`).
2. Разрешение jump:
   - читать `detour` у outbound основного тега;
   - искать outbound jump по `tag == detour` в `outbounds`;
   - кодировать jump через текущий `ShareURIFromOutbound`.
3. Ошибки/статусы:
   - `detour` пуст или отсутствует → пункт `Copy jump server link` **не показывается**;
   - jump-тег не найден в `outbounds` → понятная ошибка;
   - неподдерживаемый тип jump outbound → существующая ошибка `ErrShareURINotSupported`.

### Out of scope

- «Одна ссылка на всю цепочку» (custom URI/sbl-схема).
- Рекурсивное копирование multi-hop (jump->jump->...).
- Изменение формата share URI.

## 4. UX / Тексты

Минимально:

- `servers.menu_copy_server_link`
- `servers.menu_copy_jump_server_link`
- `servers.copy_jump_not_configured`
- `servers.copy_jump_not_found`

Пункт `Copy jump server link`:
- показывается только если у текущего outbound есть непустой `detour`;
- скрывается, если `detour` отсутствует или пуст.

## 5. Технический подход

1. **UI** (`ui/clash_api_tab.go`)
   - расширить `serversProxyContextMenu`;
   - добавить обработчик `serversRunCopyJumpShareURIToClipboard`.
2. **Config access** (`core/config/outbound_share.go` или рядом)
   - helper `GetDetourTagForOutbound(tag string) (string, error)` либо локальная логика чтения map outbound;
   - использовать существующий `GetOutboundMapByTag`.
3. **Encoding**
   - основной hop: текущий `ShareProxyURIForOutboundTag`;
   - jump hop: `ShareURIFromOutbound(jumpOutboundMap)`.

## 6. Критерии приёмки

1. Для обычной ноды без `detour`:
   - `Copy server link` работает как раньше;
   - `Copy jump server link` не отображается.
2. Для ноды с `detour`:
   - `Copy server link` возвращает URI main-hop;
   - `Copy jump server link` возвращает URI jump-hop.
3. Если `detour` указывает на несуществующий тег:
   - не panic, понятное сообщение пользователю.
4. Регрессий по текущему `Copy link` нет.

## 7. Тест-план

1. Юнит:
   - извлечение `detour` из outbound map;
   - поиск jump outbound по тегу;
   - ошибки (no detour / missing tag).
2. Интеграция (ручная):
   - нода без jump;
   - нода с SOCKS jump;
   - нода с VLESS jump;
   - jump unsupported type.

