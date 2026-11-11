# ADR-003: Підключати джерела даних через адаптери з retry, timeout і статусом Offline
Status: Accepted
Date: 2026-11-11

## Context
Дані надходять із зовнішнього API та/або сенсорів. Джерела можуть бути
недоступними, повільними або повертати некоректні дані. Збій одного джерела
не має зупиняти систему (NFR-03), нові джерела мають додаватися без змін
у ядрі (NFR-05).

## Decision Drivers
- надійність і обробка відмов;
- розширюваність;
- секрети лише в Infrastructure/конфігурації (NFR-02).

## Considered Options
1. Application напряму викликає HTTP-клієнт провайдера.
2. Порт IMeasurementSource + окремий адаптер на кожен тип джерела;
   retry/timeout у IngestionService.
3. Event-driven схема з брокером повідомлень.

## Decision
Ввести порт IMeasurementSource. Адаптери (WeatherApiAdapter,
SimulatedSourceAdapter) живуть в Infrastructure. IngestionService виконує
до 3 спроб із зростаючою затримкою, таймаут 5 с; після невдачі сенсор
позначається OFFLINE, помилка логується, обробка інших сенсорів триває.

## Rationale
Порт ізолює Application від деталей провайдера, спрощує тестування
fake-джерелом. Брокер надлишковий для MVP.

## Consequences
### Positive
- ізоляція збоїв; просте додавання джерел; тестованість відмов.
### Negative / Risks
- затримки retry збільшують час циклу опитування;
- потрібен підбір параметрів retry/timeout на практиці.

## Related
- NFR-03; NFR-05; NFR-06; FR-02; FR-07; ASR-03; ASR-05; ADR-001; Issue #<N>;
- docs/uml/collect-measurements-sequence.puml