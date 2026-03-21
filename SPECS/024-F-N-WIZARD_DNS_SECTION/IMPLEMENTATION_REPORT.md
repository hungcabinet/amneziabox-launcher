# IMPLEMENTATION_REPORT: WIZARD_DNS_SECTION (024)

## Статус: дорабатывается по мере использования

Персистентность резолвера: **только `dns_options`** в `state.json`; дубль в **`config_params`** убран, чтение старых файлов — в **`restoreDNS`** (см. **docs/WIZARD_STATE.md**).

## Сделано

- Вкладка **DNS** в визарде (`ui/wizard/tabs/dns_tab.go`), порядок табов: Sources → Outbounds → **DNS** → Rules → Preview.
- Модель: `WizardModel` — серверы (`[]json.RawMessage`), `DNSRulesText`, `DNSFinal`, `DNSStrategy`, `DNSIndependentCache`, `DefaultDomainResolver`, `DefaultDomainResolverUnset`.
- State: `WizardStateFile.DNSOptions` → JSON **`dns_options`** (`PersistedDNSState`: **`servers`**, **`rules`** — массив объектов как в sing-box `dns.rules`, **`final`**, **`strategy`**, **`independent_cache`**, резолвер…). В UI правила редактируются построчным текстом; при сохранении state парсятся в **`rules`**. Ключ **`rules_text`** в `state.json` **не** используется (не читается и не пишется). **`default_domain_resolver`** / **`default_domain_resolver_unset`** — только в **`dns_options`**; **`config_params`** для резолвера не используется (миграция со старых снимков).
- Шаблон: корневая секция **`dns_options`** в `bin/wizard_template.json` с `default_domain_resolver`; загрузчик читает её **раньше**, чем `config.route.default_domain_resolver` (`ui/wizard/template/loader.go`).
- Слияние шаблона и state: `ApplyWizardDNSTemplate`, `LoadPersistedWizardDNS`, `PersistedDNSRulesForState` — `ui/wizard/business/wizard_dns.go`; модель персистентности DNS — `ui/wizard/models/dns_state.go`.
- Сборка конфига: `buildDNSSection` / `MergeDNSSection`; `MergeRouteSection` — `default_domain_resolver` или удаление ключа.
- Документация: `docs/WIZARD_STATE.md`, `docs/CREATE_WIZARD_TEMPLATE.md`, `docs/CREATE_WIZARD_TEMPLATE_RU.md`, `docs/ARCHITECTURE.md`, `docs/release_notes/upcoming.md`.

## Тесты

- `go test ./ui/wizard/business/...`, `go build ./...`, `go vet ./...`.

## Открыто / на проверку

- Поведение при очень старых `state.json` без полей резолвера в `dns_options` — шаблон и **`ApplyWizardDNSTemplate`** / `fillDefaultDomainResolverIfEmpty`.

## Замечания

- Редактор сервера — JSON одного объекта sing-box.
- `NewWizardStateFile(..., dnsOptions *PersistedDNSState)` — в Get Free передаётся `nil`.
