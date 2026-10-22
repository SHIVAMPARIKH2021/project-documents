# Event Contracts

## 1. Fund-Etl Producer and Funds-Import-Consumer event contract

```json
{
  "specVersion": "1.0",
  "jobId": "c73a792c-5678-4a1b-9f20-91a3e6f982b1",
  "eventType": "com.sec.pipeline.fund.etl.completed",
  "timestamp": "2026-10-17T16:45:00.123Z",
  "source": "fund-etl:sec-edgar-cron",
  "correlationId": "corr-2026-q1-run-001",
  "payload": {
    "year": "2026",
    "quarter": "1",
    "error":"",
    "isForcedImport": false,
    "recordsProcessed": 1420,
    "status": "SUCCESS"
  }
}
```
#### Field Responsibilities
- ```specVersion / eventType```: Identifies the contract format and exact business event name. Enables Spring consumers to implement topic filtering or conditional handler routing.
- ```eventId```: A unique UUID generated per event. The Spring Batch consumer can store this in a processed-event table or Redis cache for idempotency (preventing duplicate batch executions if RabbitMQ re-delivers an unacknowledged message).
- ```correlationId```: Carried across both pipelines and logged with every SQL transaction for complete end-to-end auditability.
- ```payload```: Contains the parameters required to initialize Spring Batch's JobParameters (year, quarter, force, executor).

# Repository Architecture
## API - Microservices
- Below is an example structure of an api/microservice repository.
```plantext
fund-service/
├── build.gradle
├── gradlew
├── gradlew.bat
├── settings.gradle  
├── .gitignore                     # Ignore .gradle/, build/, out/
├── .editorconfig
├── .gitattributes
├── README.md
├── Dockerfile
├── .github/
│   └── workflows/
│       └── ci.yml
└── src/
    ├── main/
    │   ├── java/funds/
    │   │   ├── FundServiceApplication.java
    │   │   │
    │   │   ├── domain/                         # Pure Business Logic (No Spring/JPA dependencies)
    │   │   │   ├── model/                      # Aggregate Roots & Entities (Fund, Benchmark)
    │   │   │   ├── valueobject/                # Value Objects (Ticker, ExpenseRatio, Cusip)
    │   │   │   ├── event/                      # Domain Events (FundCreatedEvent, WeightRebalancedEvent)
    │   │   │   ├── repository/                 # Pure Java Interfaces (FundRepository.java)
    │   │   │   └── service/                    # Domain Services (Complex cross-aggregate calculations)
    │   │   │
    │   │   ├── application/                    # Use Case Orchestration
    │   │   │   ├── FundApplicationService.java # Transaction boundary, coordinates Aggregates & Repos
    │   │   │   ├── dto/                        # Command & Query DTOs (e.g., CreateFundCommand)
    │   │   │   └── event/                      # Application Event Listeners (e.g., RabbitMQ event consumers)
    │   │   │
    │   │   ├── infrastructure/                 # Frameworks, Database, and Broker Integrations
    │   │   │   ├── persistence/                # Database Implementation
    │   │   │   │   ├── entity/                 # JPA @Entity models (FundJpaEntity.java)
    │   │   │   │   ├── repository/             # Spring Data JpaRepository interfaces
    │   │   │   │   ├── adapter/                # FundRepositoryImpl (implements domain/repository/FundRepository)
    │   │   │   │   └── mapper/                 # Entity <-> Domain Model mapping functions
    │   │   │   ├── messaging/                  # RabbitMQ Publishers & Outbox dispatchers
    │   │   │   └── config/                     # Spring @Configuration (RabbitMQ, DataSource, Security)
    │   │   │
    │   │   └── api/                            # REST Delivery Mechanism
    │   │       ├── FundController.java         # @RestController endpoints
    │   │       ├── dto/                        # HTTP Request & Response payloads
    │   │       ├── mapper/                     # HTTP DTO <-> Application DTO mappers
    │   │       └── exception/                  # @RestControllerAdvice global error handlers
    │   │
    │   └── resources/                          # Standard classpath resource directory
    │       ├── application.yml                 # Shared baseline configuration
    │       ├── application-local.yml           # Local development overrides
    │       └── application-prod.yml            # Docker container deployment configuration
    │
    └── test/                                   # Standard test source root
        ├── java/funds/
        │   ├── domain/                         # Fast unit tests for Aggregates & Value Objects
        │   ├── application/                    # Mocked tests for Application Services
        │   ├── api/                            # Controller slice tests (@WebMvcTest)
        │   └── infrastructure/                 # Integration tests with Testcontainers
        └── resources/
            └── application-test.yml            # Isolated test properties
```