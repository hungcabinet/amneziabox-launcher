# SPEC: Servers — контекстное меню и share URI из config.json

Задача: на вкладке **Servers** (список прокси Clash API) дать пользователю **контекстное меню по ПКМ** на строке прокси с возможностью **скопировать share-ссылку** (формат как у строки подписки), собранную из уже записанного **config.json**, без повторной загрузки подписок и без хранения исходной строки узла.

**Статус:** закрыта (реализовано). Детали реализации — **PLAN.md**, **IMPLEMENTATION_REPORT.md**.

---

## 1. Проблема

### 1.1 До изменений

- У пользователя есть актуальный config.json с outbounds и при необходимости endpoints (WireGuard).
- В UI списка прокси (Clash API) не было способа быстро получить ссылку вида vless://, wireguard:// и т.д. для выбранного тега.
- Надёжный источник истины — тот же JSON, что использует sing-box.

### 1.2 Цель

- По **ПКМ** на строке списка открывать меню.
- Первая строка меню — информация о **типе** прокси из ответа Clash API (поле type), в **нижнем регистре**; если тип не пришёл — локализованная заглушка.
- Вторая строка — **Копировать ссылку** / **Copy link**: построить URI из outbounds по tag, совпадающему с именем прокси в API; если outbound с таким тегом нет — попытаться WireGuard в endpoints.
- Типы selector / urltest / direct не блокируют меню: пользователь может нажать Copy link; кодировщик вернёт ошибку, если тип не поддерживается.

---

## 2. Требования

### 2.1 Данные из Clash API

- При разборе GET /proxies для каждого имени из group.all читать объект прокси и поле **type** (строка).
- Сохранять в **api.ProxyInfo.ClashType**.

### 2.2 Контекстное меню (Fyne)

- Область ПКМ — вся строка списка: строка оборачивается виджетом, перехватывающим secondary tap.
- Первая строка: **ProxyInfo.ContextMenuTypeLine** — lower-case тип или локализованная заглушка.
- Чтобы текст не был серым (как у Disabled), пункт не помечать Disabled; для заголовка использовать **Action: nil** (на десктопе клик по строке не закрывает меню без действия).
- Вторая строка: Copy link — асинхронно сборка URI, буфер обмена; ошибки — диалог / локализованный текст для ErrShareURINotSupported.

### 2.3 Кодирование share URI

- Зеркало к ParseNode / buildOutbound: из map outbound (как в JSON) собрать URI для vless, vmess, trojan, shadowsocks, socks, hysteria2, ssh, wireguard.
- Для WireGuard в endpoints: **ShareURIFromWireGuardEndpoint**, один peer; несколько peers — ErrShareURINotSupported.
- Вход по пути к конфигу и тегу: **ShareProxyURIForOutboundTag** — один раз распарсить корень config.json, поиск в outbounds, при необходимости в endpoints.

### 2.4 Локализация и документация

- Ключи: servers.menu_copy_link, servers.menu_context_type_unknown, servers.copy_link_resolving, servers.copy_link_done, servers.copy_link_not_supported (en + ru).
- Описать в docs/ParserConfig.md, docs/ARCHITECTURE.md, при необходимости README, TEST_README, release_notes.

### 2.5 Критерии приёмки

- ПКМ по строке открывает меню с типом и Copy link.
- Для поддерживаемого outbound ссылка валидна и снова разбирается ParseNode (где покрыто тестами).
- WireGuard только в endpoints — ссылка строится.
- Неподдерживаемый тип — понятное сообщение.
- Тесты затронутых пакетов проходят.

---

## 3. Связанные задачи

- WIREGUARD_URI: SPECS/009-F-C-WIREGUARD_URI
- Транспорты подписки: SPECS/023-F-C-SUBSCRIPTION_TRANSPORT_VLESS_TROJAN

---

## 4. Вне scope

- Изменение формата config.json или Clash API.
- Одна ссылка на группу целиком (только один тег → один URI или ошибка).
