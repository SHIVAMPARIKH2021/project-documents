```text
====================================================================================================
AWS WORLD (SNS + SQS Distributed Pattern):
[ Producer ] ──> [ SNS Topic ] ──(Filter / Fanout)──> [ SQS Queue ] ──> [ Consumer Worker ]

RABBITMQ WORLD (1-to-1 Equivalent in a Single Lightweight Broker):
[ Producer ] ──> [ Exchange  ] ──(Binding / Routing)─> [ Queue     ] ──> [ Consumer Worker ]
====================================================================================================

+----------------------------------------------------------------------------------------------------+
|                                    User/Client Machine (Dev / CI Controller)                       |
|   - Codebase editing (IntelliJ IDEA)                                                               |
|   - Docker CLI & Remote Git pushes                                                                 |
|   - Jenkins pipeline dispatch                                                                      |
+-------------------------------------------------+--------------------------------------------------+
                                                  |
                                                  | Git Push / Jenkins SSH Runner
                                                  v
+----------------------------------------------------------------------------------------------------+
|                               WINDOWS SERVER MACHINE (Docker Desktop / Host)                       |
|                                                                                                    |
|  +-----------------------------------------------------------------------------------------------+ |
|  |                              SERVING & PRESENTATION TIER (Future)                             | |
|  |                                                                                               | |
|  |   [ Browser UI ] ──HTTP GET──> [ BFF / API Gateway ] ──SQL Queries──┐                         | |
|  |                                                                     │                         | |
|  +---------------------------------------------------------------------│-------------------------+ |
|                                                                        │                           |
|  +---------------------------------------------------------------------│-------------------------+ |
|  |                       PIPELINE & INGESTION TIER (RabbitMQ Architecture)                       | |
|  |                                                                     │                         | |
|  |   +--------------------------+                                      │                         | |
|  |   | fund-etl (Python Cron)   |                                      │                         | |
|  |   |--------------------------|                                      │                         | |
|  |   | - Computes Year & Q      |                                      │                         | |
|  |   | - ticker=True            |                                      │                         | |
|  |   | - dry_run=False          |                                      │                         | |
|  |   | - Ingests SEC EDGAR data |                                      │                         | |
|  |   +------------+-------------+                                      │                         | |
|  |                |                                                    │                         | |
|  |                | 1. Publish AMQP Message                            │                         | |
|  |                |    Routing Key: "fund.etl.completed"               │                         | |
|  |                v                                                    │                         | |
|  |   +-------------------------------------------------------+         │                         | |
|  |   |              RABBITMQ BROKER (~100 MB RAM)            |         │                         | |
|  |   |                                                       |         │                         | |
|  |   |   +-----------------------------------------------+   |         │                         | |
|  |   |   | Exchange: etl.events.exchange (Topic)         |   |         │                         | |
|  |   |   | [Equivalent to: AWS SNS Topic]                |   |         │                         | |
|  |   |   +-----------------------+-----------------------+   |         │                         | |
|  |   |                           |                                     │                         | |
|  |   |                           | 2. Binding Rule:                    │                         | |
|  |   |                           |    "fund.etl.#" (Routes message)    │                         | |
|  |   |                           v                                     │                         | |
|  |   |   +-----------------------------------------------+   |         │                         | |
|  |   |   | Queue: fund.import.trigger.queue              |   |         │                         | |
|  |   |   | [Equivalent to: AWS SQS Queue]                |   |         │                         | |
|  |   |   +-----------------------+-----------------------+   |         │                         | |
|  |   +---------------------------│---------------------------+         │                         | |
|  |                               │                                     │                         | |
|  |                               | 3. AMQP Push / Listener             │                         | |
|  |                               v                                     │                         | |
|  |   +-------------------------------------------------------+         │                         | |
|  |   | funds-import-job (Spring Batch Consumer Worker)       |         │                         | |
|  |   |-------------------------------------------------------|         │                         | |
|  |   | - @RabbitListener receives event                      |         │                         | |
|  |   | - Step 1: InitReportingQuarterTasklet (Seed Qtr)      |         │                         | |
|  |   | - Step 2: benchmarkExtractionStep (Benchmarks)        |         │                         | |
|  |   | - Step 3: fundsImportStep (Fund Master & Bridge)      |         │                         | |
|  |   +---------------------------+---------------------------+         │                         | |
|  |                               │                                     │                         | |
|  |                               | 4. Batch Persist                    │                         | |
|  |                               v                                     v                         | |
|  |   +---------------------------------------------------------------------------------------+   | |
|  |   |                             POSTGRESQL DATABASE CONTAINER                             |   | |
|  |   |   Schemas:                                                                            |   | |
|  |   |     - sec_financials (Raw SEC Submissions & Tags)                                     |   | |
|  |   |     - analytics      (reporting_quarter, fund_master, quarter_fund_association)       |   | |
|  |   |     - batch          (Spring Batch metadata executions)                               |   | |
|  |   +---------------------------------------------------------------------------------------+   | |
|  +-----------------------------------------------------------------------------------------------+ |
+----------------------------------------------------------------------------------------------------+
```