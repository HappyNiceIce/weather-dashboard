# Requirements

## Контекст
Документ містить актуалізований каталог вимог до проєкту Weather Dashboard
(панель моніторингу погодних та сенсорних даних), сформований на основі
результатів ПЗ 1-2 та ЛР1 (backlog, issues #1–#5). Вимоги використовуються
як основа для UML-моделей та архітектурного проєктування у ЛР2.

## Функціональні вимоги

| ID | Формулювання | Пріоритет | Критерій приймання | Джерело |
|----|--------------|-----------|--------------------|---------|
| FR-01 | Користувач може переглянути поточні показники (температура, вологість, тиск, швидкість вітру) | Must | Відображається список сенсорів; для кожного видно значення, одиницю виміру та час останнього оновлення | Issue #1, UC-02 View Current Readings |
| FR-02 | Система періодично збирає показники із зовнішніх джерел (погодний API, сенсори) і зберігає їх із часовою міткою | Must | Після успішного опитування вимірювання збережено з sensorId, значенням і timestamp | ПЗ, UC-01 Collect Measurements |
| FR-03 | Користувач може переглянути історію вимірювань сенсора | Must | Повертаються вимірювання, відсортовані за часом; для порожньої історії повертається порожній список | ПЗ, UC-03 View History |
| FR-04 | Користувач може відфільтрувати вимірювання за діапазоном дат | Must | Повертаються лише вимірювання з періоду; якщо дата початку пізніша за дату кінця – помилка валідації | Issue #5, UC-04 Filter Readings by Date |
| FR-05 | Система обробляє відмову джерела даних: повторює спроби, позначає джерело OFFLINE, продовжує збір з інших джерел | Must | Після 3 невдалих спроб статус = OFFLINE; після наступного успішного зчитування = ONLINE | Issue #4, UC-01 (extend) |
| FR-06 | Система позначає застарілі дані, якщо останнє вимірювання старше за заданий поріг | Should | Для вимірювання старше порога (за замовчуванням 15 хв) на панелі показано позначку «застаріле» | ПЗ, UC-02 |
| FR-07 | Система перевіряє правильність отриманих даних до збереження | Must | Значення поза допустимим діапазоном типу вимірювання відхиляється і не зберігається | ПЗ, UC-01 (include) |
| FR-08 | Адміністратор може додавати, вмикати та вимикати сенсори і джерела даних | Should | Після додавання сенсор з'являється у списку; вимкнений сенсор не опитується | ПЗ, UC-05 Manage Sensors |

## Нефункціональні вимоги

| ID | Quality Attribute | Формулювання | Пріоритет | Як перевіряється |
|----|-------------------|--------------|-----------|------------------|
| NFR-01 | Maintainability / Testability | Бізнес-логіка не залежить від UI, HTTP, SQLite та конкретного провайдера даних | Must | Unit-тести Domain/Application запускаються без БД, мережі та UI (fake-адаптери) |
| NFR-02 | Security | Ключі API та секрети не зберігаються в репозиторії | Must | .env, *.secret, secrets.json виключені через .gitignore; repository audit – немає ключів і токенів |
| NFR-03 | Reliability | Збій одного джерела не зупиняє збір з інших; до 3 повторів, таймаут запиту 5 с | Must | Тест із fake-джерелом, що кидає помилку: інші джерела оброблені, джерело → OFFLINE |
| NFR-04 | Performance | 95 % запитів поточних показників обробляються не довше 500 мс за 50 сенсорів, 100 000 вимірювань і до 20 одночасних користувачів у тестовому середовищі | Should | Вимірювання на тестовому наборі даних |
| NFR-05 | Modifiability / Integration | Додавання нового типу джерела потребує лише нового адаптера без змін Domain/Application | Should | Code review: погодний API і симульоване джерело підключаються через один порт |

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
Related requirement: NFR-03 (FR-05)
Failure of one data source must not stop the system.
Architectural impact: джерела ізольовані за портом IMeasurementSource; retry, timeout і статус OFFLINE обробляються в Application.

### ASR-04
Related requirement: NFR-04
95 % requests for current readings within 500 ms.
Architectural impact: окремий швидкий запит «останнє значення по кожному сенсору», індекс (sensorId, timestamp) у SQLite.

### ASR-05
Related requirement: NFR-05
New data source = new adapter only.
Architectural impact: порт IMeasurementSource + адаптер на кожен тип джерела.

## Constraints
- мова реалізації: C#;
- платформа: .NET 10;
- система керування версіями: Git;
- repository: GitHub (HappyNiceIce/weather-dashboard);
- локальне зберігання: SQLite;
- зовнішній погодний API – лише через адаптер, ключ не в репозиторії;
- процес розробки: спіральна модель з інкрементами (ПЗ 1-2); MVP за 6–8 тижнів;
- архітектурна документація зберігається у docs/.


## Словник предметної області

| Поняття | Призначення | Основні правила |
|---------|-------------|-----------------|
| Station | місце розташування, що об'єднує сенсори | має назву та локацію |
| Sensor | вимірювальний канал одного типу показника | має тип вимірювання та одиницю; належить одній станції; читає дані з одного DataSource |
| DataSource | зовнішнє джерело даних (погодний API або фізичний сенсор) | має тип, статус ONLINE/OFFLINE і час останнього успішного зчитування |
| Measurement | одне зафіксоване значення сенсора | має значення і timestamp; належить одному сенсору; вважається застарілим, якщо старше за поріг |
| MeasurementType | вид величини | TEMPERATURE, HUMIDITY, PRESSURE, WIND_SPEED |
| SourceType | вид джерела | WEATHER_API, PHYSICAL_SENSOR |
| SourceStatus | стан джерела | ONLINE, OFFLINE |

## Трасованість FR → Use Case

| FR | Use Case |
|----|----------|
| FR-01, FR-06 | UC-02 View Current Readings |
| FR-02 | UC-01 Collect Measurements |
| FR-03 | UC-03 View History |
| FR-04 | UC-04 Filter Readings by Date |
| FR-05 | UC-07 Handle Source Failure (extend UC-01) |
| FR-07 | UC-06 Validate Reading (include UC-01) |
| FR-08 | UC-05 Manage Sensors |


## Специфікація ключового use case

| Поле | Значення |
|------|----------|
| ID | UC-01 |
| Name | Collect Measurements |
| Primary Actor | Scheduler |
| Secondary Actor | External Data Source |
| Preconditions | Сенсор і джерело даних зареєстровані та увімкнені |
| Trigger | Спрацював таймер опитування |
| Main Scenario | 1. Scheduler запускає збір для сенсора. 2. Система запитує останнє значення в джерела. 3. Система перевіряє значення (UC-06). 4. Система зберігає Measurement з часовою міткою. 5. Система оновлює статус джерела на ONLINE. 6. Система повертає результат Success. |
| Alternative / Error | A1: значення поза допустимим діапазоном → Measurement не зберігається, результат Rejected. A2: джерело недоступне або таймаут 5 с (UC-07) → до 3 повторів; якщо не вдалося, джерело OFFLINE, помилка логується, збір з інших джерел триває. |
| Postconditions | Успіх: Measurement збережено, джерело ONLINE. Збій: нових даних немає, джерело OFFLINE. |
| Related FR/NFR | FR-02, FR-05, FR-07; NFR-01, NFR-03 |