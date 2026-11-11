# Component Interfaces

Документ описує основні контракти взаємодії між компонентами Weather Dashboard.
Інтерфейси визначають операції, вхідні та вихідні дані, помилки і пов'язані вимоги.
Контракти не залежать від конкретного UI або БД.

## IIngestionService (Presentation → Application)
Відповідальність: запускає збір показників для одного або всіх сенсорів.

| Операція | Вхід | Вихід | Помилки / умови | Вимога |
|---|---|---|---|---|
| collect | SensorId | CollectResult (Success / Rejected / Failed) | SensorNotFound; збій джерела повертається як Failed, а не виняток | FR-02, FR-05, FR-07 |
| collectAll | – | CollectResult[] | збій одного джерела не зупиняє інші; вимкнені сенсори пропускаються | NFR-03, FR-08 |

## IMeasurementSource (Application → Infrastructure, порт)
Відповідальність: отримання останнього значення з конкретного зовнішнього джерела.

| Операція | Вхід | Вихід | Помилки / умови | Вимога |
|---|---|---|---|---|
| fetchLatest | SensorDescriptor | RawReading | SourceUnavailableError, TimeoutError (5 с), InvalidResponseError | FR-02, NFR-03, NFR-05 |

Передумова: ключі API беруться з конфігурації та не логуються (NFR-02).
Постумова: адаптер не змінює стан Domain і не зберігає дані.

## IMeasurementRepository (Application → Infrastructure, порт)
Відповідальність: збереження та читання вимірювань.

| Операція | Вхід | Вихід | Помилки / умови | Вимога |
|---|---|---|---|---|
| save | Measurement | Measurement | PersistenceError | FR-02 |
| getLatestForAll | – | Measurement[] | PersistenceError | FR-01, FR-06, NFR-04 |
| listBySensor | SensorId, from?, to? | Measurement[] | PersistenceError; from ≤ to | FR-03, FR-04 |

## IDataSourceRepository (Application → Infrastructure, порт)
Відповідальність: зберігання стану джерел даних.

| Операція | Вхід | Вихід | Помилки / умови | Вимога |
|---|---|---|---|---|
| updateStatus | SourceId, SourceStatus | void | SourceNotFound; PersistenceError | FR-05 |
| getById | SourceId | DataSource? | PersistenceError | FR-05, FR-08 |

## IMonitoringService (Presentation → Application)
Відповідальність: запити користувача: поточні показники та історія.

| Операція | Вхід | Вихід | Помилки / умови | Вимога |
|---|---|---|---|---|
| getCurrentReadings | – | CurrentReadingsDto[] (значення, одиниця, час, ознака stale) | – | FR-01, FR-06 |
| getHistory | SensorId, from?, to? | MeasurementDto[] | SensorNotFound; InvalidPeriod (from > to) | FR-03, FR-04 |