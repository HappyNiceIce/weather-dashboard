# ADR-001: Використати Modular Monolith + Layered Architecture для MVP
Status: Accepted
Date: 2026-11-12

## Context
Weather Dashboard є невеликим навчальним проєктом. Система має бути зрозумілою
для розробки, підтримуваною, тестованою та простою для локального deployment.
Незалежне масштабування підсистем не є вимогою. Процес розробки – спіральна
модель з інкрементами, MVP очікується за 6–8 тижнів.

## Decision Drivers
- низька складність deployment;
- чіткий розподіл відповідальності;
- можливість тестувати бізнес-логіку окремо від UI та БД;
- невеликий розмір проєкту;
- відсутність потреби в незалежному масштабуванні;
- мінімальні операційні витрати.

## Considered Options
1. Modular Monolith + Layered Architecture.
2. Microservices (Ingestion Service, Query API).
3. Простий моноліт без явного розділення на шари.

## Decision
Використовувати Modular Monolith із шарами Presentation, Application, Domain,
Infrastructure. Залежності спрямовані: Presentation → Application → Domain;
Infrastructure реалізує порти, оголошені в Application.

## Rationale
Один застосунок не потребує інфраструктури міжсервісної взаємодії.
Шари задовольняють NFR-01 (бізнес-логіка тестується без UI та БД).
Microservices відхилено через надмірну складність для невеликого проєкту.
Простий моноліт відхилено, бо він погіршує тестованість і ускладнює заміну
джерел даних та сховища.

## Consequences
### Positive
- одна кодова база та простий deployment;
- зручне тестування бізнес-логіки;
- менша операційна складність.

### Negative / Risks
- компоненти не масштабуються незалежно;
- потрібно контролювати залежності між шарами;
- із ростом системи моноліт може ускладнитися.

## Related
- NFR-01; ASR-01;
- Issue #8;
- docs/architecture/architecture.md;
- docs/uml/components.puml