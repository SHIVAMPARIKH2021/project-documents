# 🏢 Employee Management & Finance System (EMFS)

---

# 📌 1. Overview

## Purpose

- The EMFS system centralizes employee, payroll, document, and notification management into a single platform.
- The system provides sample pay stubs during the hiring and onboarding process, enabling employees to better understand their anticipated bi-weekly or monthly earnings and compensation structure.

## Objectives

* Centralized employee & financial data
* Automated payroll & compliance
* Document expiry tracking & notifications
* Auditability & traceability
* Scalable system architecture

---

# 🧩 2. System Architecture

## High-Level Architecture

```
[ React UI ]
       ↓
[ Spring Boot APIs ]
       ↓
[ PostgreSQL Database ]
       ↓
[ Batch Jobs / Scheduler ]
```

---

## Technology Stack

| Layer       | Technology                |
| ----------- | ------------------------- |
| Frontend    | React                     |
| Backend     | Spring Boot               |
| Database    | PostgreSQL                |
| UI Testing  | Playwright (TypeScript)   |
| API Testing | RestAssured (Java)        |
| Batch       | Python / Spring Scheduler |

---

# 🗄️ 3. Database Design
```Sql
CREATE DATABASE emfs
    WITH
    OWNER = postgres
    ENCODING = 'UTF8';

CREATE ROLE emfs_admin
    LOGIN
    PASSWORD 'strong/?12x3401';

CREATE ROLE emfs_app
    LOGIN
    PASSWORD 'strong/?12x3402';
```

## Core Tables

* employees
* departments
* jobs
* salaries
* payroll
* attendance
* leaves
* employee_documents

---

## Audit Fields

```
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
modified_at TIMESTAMP NULL
created_by VARCHAR
modified_by VARCHAR
```

---

## ER Diagram

```
employees ────────┐
  │               │
  │               ▼
  │        departments ─── locations
  │
  ▼
employee_documents ─── document_type (ENUM)
  │
  ▼
payroll ─── salaries
  │
  ▼
tax_details

employees ─── attendance
employees ─── leaves
employees ─── performance_reviews
```

---

## Document Table (Key Fields)

```
document_type ENUM
status ENUM
expiry_date DATE
last_notified_at TIMESTAMP
```

---

# 🔁 ENUM Definitions

```
employee_status = ACTIVE, INACTIVE, ON_LEAVE, RESIGNED
attendance_status = PRESENT, ABSENT, WFH, HALF_DAY
leave_status = PENDING, APPROVED, REJECTED, CANCELLED
leave_type = SICK, CASUAL, EARNED, UNPAID
tax_regime = OLD, NEW
document_type = AADHAAR, PAN, PASSPORT
```

---

# 🔌 4. API Design

## Employee APIs

```
GET /employees
GET /employees/{id}
POST /employees
PUT /employees/{id}
```

---

## Document APIs

```
GET /documents/{employeeId}
POST /documents
PUT /documents/{id}
```

---

## Payroll APIs

```
GET /payroll
GET /salary-history/{employeeId}
```

---

# 🔔 5. Notification System

## Logic

```
IF expiry_date <= CURRENT_DATE + 7 days
AND (last_notified_at IS NULL OR last_notified_at < CURRENT_DATE)
→ Send Notification
```

---

## Sequence Diagram (Notification Flow)

```
Scheduler → DB: Fetch expiring documents
DB → Scheduler: Return records
Scheduler → Notification Service: Send email
Notification Service → Employee: Email sent
Scheduler → DB: Update last_notified_at
```

---

# ⚙️ 6. Batch Jobs

---

## 🐍 Python Batch (Data Generation)

```python
from faker import Faker
import pandas as pd

fake = Faker()

def generate_employees(n=1000):
    data = []
    for _ in range(n):
        data.append({
            "name": fake.name(),
            "email": fake.email(),
            "status": "ACTIVE"
        })
    return pd.DataFrame(data)
```

---

## ☕ Spring Boot Batch (Scheduler)

```java
@Scheduled(cron = "0 0 2 * * ?")
public void runExpiryJob() {
    List<Document> docs = repository.findExpiringDocs();
    
    for (Document doc : docs) {
        notificationService.send(doc);
        doc.setLastNotifiedAt(LocalDateTime.now());
    }
}
```

---

# 🧠 7. Domain Driven Design (DDD)

---

## 🧩 Bounded Contexts

### 1. Employee Management

* Employee
* Department
* Job

---

### 2. Finance

* Payroll
* Salary
* Tax

---

### 3. Document Management

* EmployeeDocument
* DocumentType
* Expiry Logic

---

### 4. Notification

* NotificationService
* Scheduler

---

## 🏗️ Layered Architecture

```
Controller → Application Service → Domain → Repository
```

---

## 🧱 Example Aggregate

### Employee Aggregate

```
Employee
 ├── Personal Details
 ├── Job Info
 ├── Documents
```

---

## 🧠 Entities vs Value Objects

| Type         | Example         |
| ------------ | --------------- |
| Entity       | Employee        |
| Value Object | Address, Salary |

---

## 🔄 Domain Services

* PayrollCalculationService
* NotificationService

---

# 🔐 8. Security

* JWT Authentication
* Role-based access:

| Role     | Access          |
| -------- | --------------- |
| ADMIN    | Full            |
| HR       | Employee + Docs |
| EMPLOYEE | Self            |

---

# 🧪 9. Testing Strategy

| Layer | Tool        |
| ----- | ----------- |
| Unit  | JUnit       |
| API   | RestAssured |
| UI    | Playwright  |

---

## Example UI Flow

```
Login → Dashboard → View Documents → Check Expiry Alert
```

---

# 📊 10. Performance

* Index on `employee_id`
* Index on `expiry_date`
* Pagination on APIs

---

# 🚀 11. Future Enhancements

* Microservices architecture
* Redis caching
* Kafka-based notifications
* Audit history tables

---

# 🧠 12. Design Principles

* Separation of concerns
* Audit vs Business fields separation
* Scalable DB design
* API-first approach
* Testability via time abstraction

---

# ✅ Conclusion

This system is designed as a **production-ready, scalable, and maintainable enterprise application** with:

* Clean architecture
* Robust database design
* Strong testing strategy
* Real-world business workflows

---
