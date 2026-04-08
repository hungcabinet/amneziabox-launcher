# TASKS: 033 — SUBSCRIPTION_XRAY_JSON_ARRAY

## Этап 0 — согласование с 016 (референс)

- [ ] **016** закрыта без кода: правку `decoder.go` и ветку `[` делать **в рамках 033** (см. **016-F-C IMPLEMENTATION_REPORT**).
- [ ] Порядок: сначала общий JSON-массив, внутри — классификатор sing-box vs Xray.

## Этап 1 — модель и генерация

- [ ] Расширить `configtypes.ParsedNode` (или согласованную структуру) полями для **jump** (SOCKS) при наличии цепочки.
- [ ] Добавить в `GenerateNodeJSON` (или вспомогательную функцию, вызываемую из `GenerateOutboundsFromParserConfig`) поддержку **`detour`** и при `jump` — **две** JSON-строки в правильном порядке.
- [ ] Убедиться, что селекторы перечисляют только теги **основных** нод (jump не дублирует строки в списке серверов Clash API — уточнить по текущему коду отображения).

## Этап 2 — парсер Xray элемента

- [ ] Реализовать разбор одного элемента массива: `outbounds`, индекс по `tag`, выбор основного VLESS по **PLAN §3**.
- [ ] Реализовать извлечение SOCKS jump по `dialerProxy` и маппинг в sing-box `socks` outbound map.
- [ ] Реализовать конвертацию VLESS (`vnext`, `streamSettings`, reality) → поля, совместимые с `GenerateNodeJSON`.

## Этап 3 — интеграция подписки

- [ ] Подключить парсер в `LoadNodesFromSource` при теле `[...]` после декодера.
- [ ] Подключить ту же ветку в `ui/wizard/business/parser.go` (CheckURL, подсчёт нод).
- [ ] Применить `MakeTagUnique`, TagPrefix/Postfix/Mask, `MaxNodesPerSubscription`, Skip — как у обычных подписок.

## Этап 4 — Share URI и границы

- [ ] Определить поведение «Копировать ссылку» для chained-нод: `ErrShareURINotSupported` или документированное ограничение.
- [ ] При необходимости обновить `share_uri_encode.go` / тесты на явный отказ.

## Этап 5 — тесты и документация

- [ ] Юнит-тесты: анонимизированный JSON-массив (≥2 элемента, разные jump); проверка наличия `detour` и разных SOCKS `server`.
- [ ] `go test ./...`, `go vet ./...`, `go build ./...`.
- [ ] Обновить `docs/ParserConfig.md`, `docs/release_notes/upcoming.md`.
- [ ] Заполнить `IMPLEMENTATION_REPORT.md` по закрытию задачи.
