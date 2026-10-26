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
.
├── HELP.md                                                         # Spring Boot help & reference links
├── ReadMe.md                                                       # Project documentation, setup, and run guide
├── build.gradle                                                    # Gradle build configuration and dependencies
├── gradle/
│   └── wrapper/                                                    # Gradle wrapper files for reproducible builds
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew                                                         # Unix execution wrapper script
├── gradlew.bat                                                     # Windows execution wrapper batch script
├── settings.gradle                                                 # Gradle multi-project / root project definitions
└── src/
    ├── main/
    │   ├── java/sec/financials/
    │   │   ├── FundServiceApplication.java                         # Spring Boot application entry point (@SpringBootApplication)
    │   │   │
    │   │   ├── api/                                                # Primary/Inbound Driving Adapter (Presentation Layer)
    │   │   │   ├── controller/                                     # REST controllers handling HTTP requests and status codes
    │   │   │   │   ├── BenchmarkController.java                    # Exposes REST endpoints for Benchmark resources
    │   │   │   │   └── FundController.java                         # Exposes REST endpoints for Fund resources
    │   │   │   ├── dto/                                            # Web transport contracts (HTTP Request & Response payloads)
    │   │   │   │   ├── BenchmarkRequestDto.java                    # Inbound HTTP JSON payload for benchmark operations
    │   │   │   │   ├── BenchmarkResponseDto.java                   # Outbound HTTP JSON payload for benchmark queries
    │   │   │   │   ├── FundRequestDto.java                         # Inbound HTTP JSON payload for fund operations
    │   │   │   │   └── FundResponseDto.java                        # Outbound HTTP JSON payload for fund queries
    │   │   │   ├── exception/                                      # Global web-tier error interception & response formatting
    │   │   │   │   └── GlobalExceptionHandler.java                 # Translates domain/app exceptions to RFC 7807/HTTP status codes
    │   │   │   ├── mapper/                                         # Boundary translation between Web DTOs & Application models
    │   │   │   │   ├── BenchmarkApiMapper.java                     # Maps Benchmark HTTP DTOs <-> Benchmark Application DTOs
    │   │   │   │   └── FundApiMapper.java                          # Maps Fund HTTP DTOs <-> Fund Application DTOs
    │   │   │   └── route/                                          # Centralized HTTP URL routing and path constants
    │   │   │       └── ApiEndPoints.java                           # Immutable endpoint URI constants (e.g., base paths, versions)
    │   │   │
    │   │   ├── application/                                        # Use Case Orchestration Layer (Application Services)
    │   │   │   ├── dto/                                            # Cross-layer data carriers decoupled from transport & persistence
    │   │   │   │   ├── BaseDto.java                                # Shared audit/metadata base DTO contract
    │   │   │   │   ├── BenchmarkDto.java                           # Application-level benchmark transfer model
    │   │   │   │   └── FundDto.java                                # Application-level fund transfer model
    │   │   │   └── service/                                        # Application use case workflows & transaction management
    │   │   │       ├── BenchmarkService.java                       # Orchestrates benchmark use cases & coordinates repositories
    │   │   │       └── FundService.java                            # Orchestrates fund lifecycle workflows & transaction boundaries
    │   │   │
    │   │   ├── domain/                                             # Pure Business Logic Core (Framework-free, zero Spring/JPA)
    │   │   │   ├── benchmark/                                      # Benchmark Bounded Feature / Aggregate Slice
    │   │   │   │   ├── model/                                      # Benchmark Domain Entities & Aggregate Roots
    │   │   │   │   │   └── Benchmark.java                          # Benchmark Aggregate Root (enforces invariants & identity)
    │   │   │   │   ├── repository/                                 # Pure Java Outbound Ports (Domain Repository Interfaces)
    │   │   │   │   │   └── BenchmarkRepository.java                # Domain contract for Benchmark persistence operations
    │   │   │   │   └── valueobject/                                # Immutable Value Objects with self-validation
    │   │   │   │       └── BenchmarkMetadata.java                  # Benchmark provider, asset class, and regulatory metadata
    │   │   │   ├── fund/                                           # Fund Bounded Feature / Aggregate Slice
    │   │   │   │   ├── model/                                      # Fund Domain Entities & Aggregate Roots
    │   │   │   │   │   └── Fund.java                               # Fund Aggregate Root (enforces portfolio & fund business invariants)
    │   │   │   │   ├── repository/                                 # Pure Java Outbound Ports (Domain Repository Interfaces)
    │   │   │   │   │   └── FundRepository.java                     # Domain contract for Fund persistence operations
    │   │   │   │   └── valueobject/                                # Immutable Value Objects with self-validation
    │   │   │   │       └── Ticker.java                             # Validated ticker symbol value object
    │   │   │   └── exception/                                      # Core domain-specific business rule violation exceptions
    │   │   │       ├── BenchmarkAbsentException.java               # Thrown when a required benchmark aggregate does not exist
    │   │   │       └── FundAbsentException.java                    # Thrown when a requested fund aggregate is not found
    │   │   │
    │   │   └── infrastructure/                                     # Secondary/Outbound Driven Adapters (Technical Implementations)
    │   │       ├── config/                                         # Spring framework configuration classes
    │   │       │   ├── OpenApiConfig.java                          # Swagger/OpenAPI documentation bean definitions
    │   │       │   └── PersistenceConfig.java                      # JPA, TransactionManager, and Auditing configurations
    │   │       └── persistence/                                    # Relational database infrastructure implementation
    │   │           ├── benchmark/                                  # Benchmark persistence slice (Table models & Adapters)
    │   │           │   ├── BenchmarkJpaEntity.java                 # JPA @Entity mapping to database table (surrogate keys, columns)
    │   │           │   ├── BenchmarkRepositoryAdapter.java         # Adapter implementing domain BenchmarkRepository port
    │   │           │   └── SpringDataBenchmarkRepository.java      # Spring Data JpaRepository for Benchmark database queries
    │   │           ├── fund/                                       # Fund persistence slice (Table models & Adapters)
    │   │           │   ├── FundJpaEntity.java                      # JPA @Entity mapping to analytics.fund_master table
    │   │           │   ├── FundRepositoryAdapter.java              # Adapter implementing domain FundRepository port
    │   │           │   └── SpringDataFundRepository.java           # Spring Data JpaRepository for Fund database queries
    │   │           └── mapper/                                     # ORM translation between JPA Entities & Domain Aggregates
    │   │               ├── BenchmarkPersistenceMapper.java         # Maps BenchmarkJpaEntity <-> Benchmark domain aggregate
    │   │               └── FundPersistenceMapper.java              # Maps FundJpaEntity <-> Fund domain aggregate
    │   │
    │   └── resources/                                              # Runtime configuration and application assets
    │       ├── application.yaml                                    # Baseline configuration (DB connections, port, profiles)
    │       └── application-local.yaml                              # Local developer profile overrides (Testcontainers, debug logs)
    │
    └── test/                                                       # Test Suite mirroring production layer hierarchy
        └── java/sec/financials/
            ├── FundServiceApplicationTests.java                    # Context loads integration verification test
            │
            ├── api/                                                # Presentation-tier tests
            │   ├── controller/                                     # Web slice tests (@WebMvcTest mocking application services)
            │   │   ├── BenchmarkControllerTest.java                # HTTP validation, status code, & serialization tests for benchmarks
            │   │   └── FundControllerTest.java                     # HTTP validation, status code, & serialization tests for funds
            │   └── exception/                                      # Controller advice validation tests
            │
            ├── application/                                        # Application-tier unit & orchestration tests
            │   ├── dto/                                            # Serialization/deserialization and immutability tests for DTOs
            │   │   ├── BenchmarkRequestDtoTest.java
            │   │   ├── BenchmarkResponseDtoTest.java
            │   │   ├── FundRequestDtoTest.java
            │   │   └── FundResponseDtoTest.java
            │   └── service/                                        # Application service tests (mocking domain repositories)
            │       ├── BenchmarkServiceTest.java                   # Orchestration, mapping, and use-case verification for benchmarks
            │       └── FundServiceTest.java                        # Orchestration, mapping, and use-case verification for funds
            │
            ├── domain/                                             # Fast, framework-free unit tests for core business rules
            │   ├── benchmark/                                      # Benchmark aggregate and domain logic verification
            │   │   ├── model/
            │   │   │   └── BenchmarkTest.java                      # Aggregate business invariant and lifecycle tests
            │   │   ├── repository/
            │   │   │   └── BenchmarkRepositoryTest.java            # Contract tests for BenchmarkRepository interfaces
            │   │   └── valueobject/
            │   │       └── BenchmarkMetadataTest.java              # Value object equality, immutability, and validation tests
            │   └── fund/                                           # Fund aggregate and domain logic verification
            │       ├── model/
            │       │   └── FundTest.java                           # Aggregate business invariant and mutation tests
            │       ├── repository/
            │       │   └── FundRepositoryTest.java                 # Contract tests for FundRepository interfaces
            │       └── valueobject/
            │           └── TickerTest.java                         # Ticker formatting, character constraints, and equality tests
            │
            └── infrastructure/                                     # Integration tests for external infrastructure components
                ├── config/                                         # Spring context and configuration verification tests
                └── persistence/                                    # Database integration tests (@DataJpaTest / Testcontainers)
```