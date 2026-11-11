# ADR-002: Відокремити persistence через Repository / Port
Status: Accepted
Date: 2026-11-12

## Context
Бізнес-логіка не повинна залежати безпосередньо від SQLite або конкретної
бібліотеки доступу до даних. Це пов'язано з NFR-01 (Maintainability / Testability)
та NFR-04 (оптимізація запитів має бути ізольована в Infrastructure).

## Decision Drivers
- тестування Application/Domain без реальної БД;
- зменшення coupling;
- можливість заміни способу зберігання;
- чіткі межі між логікою та Infrastructure.

## Considered Options
1. IngestionService напряму використовує SQLite.
2. Application залежить від IMeasurementRepository, ISensorRepository, IDataSourceRepository.
3. Глобальний статичний клас доступу до даних.

## Decision
Порти (інтерфейси) оголошуються в Application. Реалізації
(SqliteMeasurementRepository та інші) знаходяться в Infrastructure.
Application залежить від контракту, а не від реалізації.

## Rationale
Залежність від абстракції ізолює бізнес-логіку від технології зберігання
і дозволяє підставити in-memory реалізацію в тестах.

## Consequences
### Positive
- нижчий coupling;
- вища тестованість;
- простіша заміна сховища;
- чіткіша архітектурна межа.

### Negative / Risks
- додаткові інтерфейси та класи;
- більше коду для простих операцій.

## Related
- NFR-01; NFR-04; ASR-01; ASR-04; ADR-001;
- Issue #8;
- docs/uml/components.puml;
- docs/architecture/architecture.md