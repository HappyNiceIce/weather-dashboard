# Requirements

## Контекст
Документ містить актуалізований каталог вимог до проєкту Weather Monitor
(панель моніторингу погодних та сенсорних даних), сформований на основі
результатів ПЗ2. Вимоги використовуються як основа для UML-моделей
та архітектурного проєктування у ЛР2.

## Функціональні вимоги

| ID | Формулювання | Пріоритет | Критерій приймання | Джерело |
|----|--------------|-----------|--------------------|---------|
| FR-01 | Оператор може зареєструвати сенсор (назва, тип вимірювання, одиниця, тип джерела) | Must | Після валідного запиту сенсор з'являється у списку; порожня назва або невідомий тип вимірювання відхиляються | UC-02 Register Sensor |
| FR-02 | Система періодично збирає показники з джерела (зовнішній API або сенсор) і зберігає їх із часовою міткою | Must | Після успішного опитування вимірювання збережено з sensorId, значенням і timestamp | UC-01 Collect Measurements |
| FR-03 | Оператор може переглянути панель з останніми значеннями всіх сенсорів | Must | Для кожного сенсора показано останнє значення і час; для Offline-сенсора – позначку Offline | UC-03 View Dashboard |
| FR-04 | Оператор може переглянути історію вимірювань сенсора за обраний період | Must | Повертаються лише вимірювання з періоду, відсортовані за часом; порожній період → порожній список; from > to відхиляється | UC-04 View History |
| FR-05 | Оператор може налаштувати правило сповіщення (умова above/below, поріг, severity) для сенсора | Should | Правило зберігається; правило без порога відхиляється | UC-05 Configure Alert Rule |
| FR-06 | Система створює Alert, якщо вимірювання порушує правило | Must | Значення, що порушує правило → створено один Alert; значення в нормі → Alert не створюється | UC-01 (include UC-07 Evaluate Alert Rules) |
| FR-07 | Система позначає сенсор Offline, якщо джерело недоступне після повторних спроб | Must | Після 3 невдалих спроб статус = OFFLINE; після наступного успішного зчитування = ONLINE | UC-01, альтернативний сценарій |
| FR-08 | Оператор може переглянути список сповіщень | Should | Список містить сенсор, час, severity, значення; відсортований від новіших | UC-06 View Alerts |

## Нефункціональні вимоги

| ID | Quality Attribute | Формулювання | Пріоритет | Як перевіряється |
|----|-------------------|--------------|-----------|------------------|
| NFR-01 | Maintainability / Testability | Domain і Application не залежать від UI, HTTP, SQLite та конкретного провайдера даних | Must | Unit-тести Domain/Application запускаються без БД, мережі та UI (fake-адаптери) |
| NFR-02 | Security | API-ключі та секрети не зберігаються в репозиторії | Must | Локальна конфігурація в .gitignore; audit репозиторію – немає ключів/токенів |
| NFR-03 | Reliability | Збій одного джерела не зупиняє збір з інших; до 3 повторів із зростаючою затримкою; таймаут запиту 5 с | Must | Тест із fake-джерелом, що кидає помилку: інші сенсори оброблені, помилка залогована, сенсор → OFFLINE |
| NFR-04 | Performance | 95 % запитів панелі (FR-03) обробляються не довше 500 мс за 50 сенсорів і 100 000 вимірювань у SQLite, до 20 одночасних користувачів, тестове середовище | Should | Вимірювання на тестовому наборі даних |
| NFR-05 | Modifiability / Integration | Додавання нового типу джерела потребує лише нового адаптера, без змін Domain/Application | Should | Code review: WeatherApiSource і SimulatedSource підключаються через один IMeasurementSource |
| NFR-06 | Observability | Кожна невдала спроба збору логується (sensorId, час, тип помилки) | Should | Тест/перегляд логів |

## Architecture-Significant Requirements

### ASR-01
Related requirement: NFR-01
Business logic must be independent of UI, persistence and data providers.
Architectural impact: розділити Presentation, Application, Domain та Infrastructure; залежності через порти.

### ASR-02
Related requirement: NFR-02
Secrets must not be stored in the repository.
Architectural impact: конфігурація та секрети виносяться назовні (змінні середовища / локальний файл поза Git).

### ASR-03
Related requirement: NFR-03 (FR-07)
Failure of one source must not stop the system.
Architectural impact: джерела ізольовані за портом IMeasurementSource; retry/timeout/статус Offline обробляються в Application.

### ASR-04
Related requirement: NFR-04
95 % dashboard requests within 500 ms.
Architectural impact: окремий швидкий запит «останнє значення по кожному сенсору», індекс (sensorId, timestamp) у SQLite.

### ASR-05
Related requirement: NFR-05
New data source = new adapter only.
Architectural impact: порт IMeasurementSource + реєстрація адаптерів за типом джерела.

## Constraints
- мова реалізації: C#;
- платформа: .NET 10;
- система керування версіями: Git;
- repository: GitHub;
- локальне зберігання: SQLite;
- зовнішній weather API – лише через адаптер, ключ не в репозиторії;
- архітектурна документація зберігається у docs/.

## Трасованість FR → Use Case
FR-01 – Register Sensor; FR-02 – Collect Measurements; FR-03 – View Dashboard;
FR-04 – View History; FR-05 – Configure Alert Rule; FR-06 – Evaluate Alert Rules;
FR-07 – Collect Measurements (alt); FR-08 – View Alerts.


## Словник предметної області

| Поняття | Призначення | Основні правила |
|---------|-------------|-----------------|
| Station | місце розташування, що об'єднує сенсори | має назву та локацію |
| Sensor | джерело показників одного типу | має тип вимірювання, одиницю, тип джерела та статус (ONLINE/OFFLINE) |
| Measurement | одне зафіксоване значення сенсора | має значення і timestamp; належить одному сенсору |
| AlertRule | умова порушення для сенсора | має умову (ABOVE/BELOW), поріг і severity |
| Alert | факт порушення правила | створюється з конкретного Measurement і AlertRule |
| MeasurementType | вид величини | TEMPERATURE, HUMIDITY, PRESSURE, WIND_SPEED |
| SensorStatus | стан сенсора | ONLINE, OFFLINE |
| Severity | рівень сповіщення | INFO, WARNING, CRITICAL |