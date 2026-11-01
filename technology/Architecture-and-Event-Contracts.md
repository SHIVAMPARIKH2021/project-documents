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
├── HELP.md
├── ReadMe.md
├── build.gradle
├── gradle
│ └── wrapper
│     ├── gradle-wrapper.jar
│     └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │ ├── java
    │ │ └── sec
    │ │     └── financials
    │ │         ├── FundServiceApplication.java
    │ │         ├── benchmark
    │ │         │ ├── domain
    │ │         │ │ ├── Benchmark.java
    │ │         │ │ └── BenchmarkRepository.java
    │ │         │ ├── persistence
    │ │         │ │ ├── BenchmarkJpaEntity.java
    │ │         │ │ ├── BenchmarkPersistenceMapper.java
    │ │         │ │ ├── BenchmarkRepositoryAdapter.java
    │ │         │ │ └── JpaBenchmarkRepository.java
    │ │         │ ├── service
    │ │         │ │ └── BenchmarkService.java
    │ │         │ └── web
    │ │         │     ├── BenchmarkController.java
    │ │         │     ├── BenchmarkRequestDto.java
    │ │         │     └── BenchmarkResponseDto.java
    │ │         ├── config
    │ │         │ └── OpenApiConfig.java
    │ │         ├── exception
    │ │         │ └── GlobalExceptionHandler.java
    │ │         └── fund
    │ │             ├── domain
    │ │             │ ├── Fund.java
    │ │             │ ├── FundBenchmarkAssociation.java
    │ │             │ └── FundRepository.java
    │ │             ├── persistence
    │ │             │ ├── FundBenchmarkAssociationJpaEntity.java
    │ │             │ ├── FundJpaEntity.java
    │ │             │ ├── FundPersistenceMapper.java
    │ │             │ ├── FundRepositoryAdapter.java
    │ │             │ └── JpaFundRepository.java
    │ │             ├── service
    │ │             │ └── FundService.java
    │ │             └── web
    │ │                 ├── FundController.java
    │ │                 ├── FundRequestDto.java
    │ │                 └── FundResponseDto.java
    │ └── resources
    │     ├── application-local.yaml
    │     └── application.yaml
    └── test
        └── java
            └── sec
                └── financials
                    ├── FundServiceApplicationTests.java
                    ├── benchmark
                    │ ├── domain
                    │ │ └── BenchmarkTest.java
                    │ ├── persistence
                    │ │ ├── BenchmarkRepositoryAdapterTest.java
                    │ │ └── BenchmarkRepositoryTest.java
                    │ ├── service
                    │ │ └── BenchmarkServiceTest.java
                    │ └── web
                    │     └── BenchmarkControllerTest.java
                    └── fund
                        ├── domain
                        │ └── FundTest.java
                        ├── persistence
                        │ ├── FundRepositoryAdapterTest.java
                        │ └── FundRepositoryTest.java
                        ├── service
                        │ └── FundServiceTest.java
                        └── web
                            └── FundControllerTest.java
```