### USe Different Ports 
Mapping different host ports to your containerized APIs is the standard way to test individual microservices directly via Postman.
Here is how port mapping works, which ports you can use, and how to set it up.

---

### Which Port Numbers Can You Use?

Port numbers range from **0 to 65535**. They are divided into three standard ranges:

* **0 – 1023 (Well-Known / System Ports):** Reserved for OS services (e.g., `80` HTTP, `443` HTTPS, `22` SSH). Requires root/admin permissions to bind on host machines. **Avoid for custom APIs.**
* **1024 – 49151 (Registered / User Ports):** Best choice for your custom APIs, databases, and message brokers.
* **49152 – 65535 (Dynamic / Ephemeral Ports):** Used temporarily by the OS for outbound sockets. Generally avoid hardcoding services here.

#### Recommended Port Allocation for Your Setup

| Service | Container Internal Port | Host Port (for Postman / Browser) | Note |
| --- | --- | --- | --- |
| **UI (Frontend)** | `3000` | `3000` | React / Next.js / Vue default |
| **BFF (Gateway)** | `4000` | `4000` | GraphQL or REST aggregation |
| **User API** | `8080` | `8081` | First domain service |
| **Order API** | `8080` | `8082` | Second domain service |
| **Payment API** | `8080` | `8083` | Third domain service |
| **PostgreSQL** | `5432` | `5432` | Avoids port collision if local Postgres isn't running |
| **Redis** | `6379` | `6379` | Caching / Sessions |
| **RabbitMQ** | `15672` | `15672` | RabbitMQ Management UI |

> **Crucial Concept: Host Port vs. Container Port**
> Inside its own container, **every API can listen on the exact same port (e.g., `8080`)** because containers have isolated network namespaces. You only differentiate them on your laptop (the host) when mapping them:
> `-p <Host_Port>:<Container_Port>`  $\rightarrow$  `-p 8081:8080`

---

### How to Run and Test with Docker

#### Option 1: Using Standalone Docker Commands

1. **Build each API image:**
```bash
docker build -t user-api ./services/user-service
docker build -t order-api ./services/order-service

```


2. **Run the containers with unique host ports:**
```bash
# User API mapped to host port 8081
docker run -d --name user-service -p 8081:8080 user-api

# Order API mapped to host port 8082
docker run -d --name order-service -p 8082:8080 order-api

```



#### Option 2: Using Docker Compose (Recommended)

Instead of running long `docker run` commands manually, define them in a `docker-compose.yml`:

```yaml
services:
  user-api:
    build: ./services/user-service
    ports:
      - "8081:8080" # http://localhost:8081 in Postman

  order-api:
    build: ./services/order-service
    ports:
      - "8082:8080" # http://localhost:8082 in Postman

  payment-api:
    build: ./services/payment-service
    ports:
      - "8083:8080" # http://localhost:8083 in Postman

```

Run everything with one command:

```bash
docker compose up -d

```

---

### Hitting Endpoints via Postman

Once the containers are running:

* **If Docker runs on your same laptop:**
* User API: `http://localhost:8081/api/v1/users`
* Order API: `http://localhost:8082/api/v1/orders`
* Payment API: `http://localhost:8083/api/v1/payments`


* **If Docker runs on your second laptop (IP: `192.168.1.50`):**
* User API: `[http://192.168.1.50:8081/api/v1/users](http://192.168.1.50:8081/api/v1/users)`
* Order API: `[http://192.168.1.50:8082/api/v1/orders](http://192.168.1.50:8082/api/v1/orders)`



---