# Architecture

## 1. Context and Scope
Weather Dashboard – навчальна інформаційна система моніторингу погодних та
сенсорних даних. Користувачі переглядають поточні показники та історію,
адміністратор керує сенсорами і джерелами. Дані надходять із зовнішнього
погодного API та фізичних сенсорів через періодичне опитування.

## 2. Architecture Drivers

| Driver | Вплив на архітектуру |
|--------|----------------------|
| Maintainability / Testability (ASR-01) | розділити UI, application logic, domain і persistence |
| Security (ASR-02) | секрети та ключі API поза репозиторієм |
| Reliability (ASR-03) | ізолювати джерела даних; retry, timeout, статус OFFLINE |
| Performance (ASR-04) | швидкий запит «останні значення», індекс по (sensorId, timestamp) |
| Integration / Modifiability (ASR-05) | нові джерела додаються як адаптери |
| Spiral + incremental process (ПЗ 1-2) | архітектура має дозволяти поступове додавання функцій і ранню перевірку ризиків інтеграції |
| Small project size | не потрібна distributed architecture |
| SQLite constraint | persistence реалізується локально |

## 3. Architecture Alternatives

### A1 – Modular Monolith + Layers (+ Ports & Adapters)
Переваги:
- проста розробка і deployment;
- одна кодова база;
- низька операційна складність;
- чіткі межі між шарами;
- адаптери ізолюють зовнішні джерела.

Недоліки:
- компоненти не масштабуються незалежно;
- потрібно контролювати внутрішні залежності.

### A2 – Microservices (Ingestion Service, Query API)
Переваги:
- незалежний deployment і масштабування збору даних;
- ізоляція збоїв між сервісами.

Недоліки:
- значно більша складність;
- міжсервісна взаємодія, складніші deployment, logging, configuration;
- надмірність для навчального проєкту.

### A3 – Event-driven (брокер повідомлень між збором і обробкою)
Переваги:
- буферизація при збоях джерел;
- слабка зв'язність.

Недоліки:
- потрібна окрема інфраструктура брокера;
- надмірна складність для MVP.

## 4. Chosen Architecture
Обрано Modular Monolith + Layered Architecture. Джерела даних і сховище
підключаються через порти (інтерфейси), реалізації знаходяться в Infrastructure.

Обґрунтування: система невелика, не потребує незалежного масштабування,
розробляється як єдиний застосунок. Ports & Adapters ізолює головний ризик
проєкту (нестабільні зовнішні джерела) від бізнес-логіки.

## 5. Components and Responsibilities

| Component | Responsibility |
|-----------|----------------|
| Presentation | Web API / UI панелі; IngestionScheduler (запускає збір за таймером) |
| Application | use cases: IngestionService, MonitoringService, SensorAdminService; retry/timeout-політика; валідація показників; DTO |
| Domain | Station, Sensor, DataSource, Measurement; правила допустимих діапазонів, застарілості даних, зміни статусу джерела |
| Infrastructure | SQLite-репозиторії, WeatherApiAdapter, PhysicalSensorAdapter, конфігурація |
| SQLite | локальне сховище даних |
| External Data Source | погодний API та фізичні сенсори (лише через адаптер) |

## 6. Dependencies
Presentation → Application → Domain.

Infrastructure реалізує порти (IMeasurementSource, IMeasurementRepository,
ISensorRepository, IDataSourceRepository), оголошені в Application.
Domain не залежить від жодного шару.

Приклад: IngestionService → IMeasurementSource ← WeatherApiAdapter.

## 7. Data Storage
Локальний SQLite (Microsoft.Data.Sqlite). Індекс по (sensorId, timestamp).

## 8. Quality Attributes
maintainability; testability; security; reliability; performance; modifiability.

## 9. Constraints
C#; .NET 10; SQLite; Git; GitHub; документація в docs/.

## 10. Risks and Open Questions
- зовнішній API може мати ліміти запитів і нестабільний формат відповіді;
- продуктивність SQLite при великому обсязі вимірювань потрібно перевірити;
- параметри retry/timeout треба підтвердити експериментально (цикл 1 спіральної моделі);
- автентифікація адміністратора ще не деталізована.

## Requirement to Architecture Mapping

| Requirement | Architectural Decision |
|-------------|------------------------|
| NFR-01 | Layered separation, порти (ADR-001, ADR-002) |
| NFR-02 | Externalized configuration/secrets |
| NFR-03, FR-05 | IMeasurementSource + retry/timeout в Application (ADR-003) |
| NFR-04 | Окремий запит latest-per-sensor + індекс |
| NFR-05 | Адаптери джерел (ADR-003) |
| FR-02 | Scheduler → IngestionService → IMeasurementSource → Repository |
| FR-07 | Валідація показників в Application/Domain перед збереженням |

## Decisions
- ADR-001 – Modular Monolith + Layered Architecture;
- ADR-002 – Repository / Port for Persistence;
- ADR-003 – Source Adapters with Resilience (retry, timeout, OFFLINE).