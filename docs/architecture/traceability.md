# Traceability Matrix

| Requirement | Use Case / UML | Component | ADR | Interface | Як перевіряється |
|---|---|---|---|---|---|
| FR-01 | UC-02 View Current Readings | Presentation, Application, Infrastructure | ADR-002 | IMonitoringService, IMeasurementRepository | test getLatestForAll |
| FR-02 | UC-01 Collect Measurements; Collect Measurements Sequence | Presentation (Scheduler), Application, Infrastructure | ADR-002, ADR-003 | IIngestionService, IMeasurementSource, IMeasurementRepository | integration test з fake-джерелом |
| FR-03 | UC-03 View History | Application, Infrastructure | ADR-002 | IMonitoringService, IMeasurementRepository | test listBySensor |
| FR-04 | UC-04 Filter Readings by Date | Application, Infrastructure | ADR-002 | IMonitoringService, IMeasurementRepository | test фільтра та from > to |
| FR-05 | UC-07 Handle Source Failure; Sequence (alt) | Application, Infrastructure | ADR-003 | IIngestionService, IMeasurementSource, IDataSourceRepository | test: 3 збої → OFFLINE |
| FR-06 | UC-02 View Current Readings | Domain, Application | ADR-001 | IMonitoringService | domain rule test (застарілість) |
| FR-07 | UC-06 Validate Reading; Sequence (Rejected) | Domain, Application | ADR-001 | IIngestionService | test значення поза діапазоном |
| FR-08 | UC-05 Manage Sensors | Presentation, Application, Infrastructure | ADR-002 | IDataSourceRepository | test вимкнений сенсор не опитується |
| NFR-01 | Component Diagram | Application, Domain, Infrastructure | ADR-001, ADR-002 | IMeasurementRepository | unit tests без SQLite |
| NFR-02 | Architecture / Configuration; .gitignore | Infrastructure | – | IMeasurementSource | repository audit |
| NFR-03 | Sequence (alt branch) | Application, Infrastructure | ADR-003 | IMeasurementSource | тест відмови одного джерела |
| NFR-04 | Component Diagram | Infrastructure | ADR-002 | IMeasurementRepository | вимірювання на тестових даних |
| NFR-05 | Component Diagram | Infrastructure | ADR-003 | IMeasurementSource | code review адаптерів |