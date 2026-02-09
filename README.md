# Patient Management

Microservices-based patient management system (Spring Boot) with API Gateway, JWT auth, gRPC, and Kafka.

## Overview
- API Gateway centralizes HTTP traffic and applies JWT validation.
- Auth Service handles authentication and token validation.
- Patient Service exposes patient CRUD, creates billing accounts via gRPC, and publishes Kafka events.
- Billing Service exposes a gRPC service for account creation.
- Analytics Service consumes Kafka events for analytics.

## Docker environment variables
Below is the set of environment variables used for containerized runs (per service).

Auth Service
```env
JWT_SECRET=tn4ATHSnCDsvYMURyLuB4aCspzHD3Ti2uaLkwP4TfSz
SPRING_DATASOURCE_URL=jdbc:postgresql://auth-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_DATASOURCE_PASSWORD=password
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_SQL_INIT_MODE=always
```

Patient Service
```env
BILLING_SERVICE_ADDRESS=billing-service
BILLING_SERVICE_GRPC_PORT=9001
SPRING_DATASOURCE_URL=jdbc:postgresql://patient-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_DATASOURCE_PASSWORD=password
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_SQL_INIT_MODE=always
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
```

API Gateway
```env
AUTH_SERVICE_URL=http://auth-service:4005
```

Analytics Service
```env
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
```

Postgres (auth-service-db, patient-service-db)
```env
POSTGRES_DB=db
POSTGRES_USER=admin_user
POSTGRES_PASSWORD=password
```

Kafka (example container config)
```env
KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093
KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
KAFKA_CFG_NODE_ID=0
KAFKA_CFG_PROCESS_ROLES=controller,broker
```

## Local startup (suggested)
1) Start billing-service (gRPC)
2) Start auth-service
3) Start patient-service
4) Start api-gateway
5) Start analytics-service

Examples:
```bash
cd auth-service && ./mvnw spring-boot:run
cd patient-service && ./mvnw spring-boot:run
cd billing-service && ./mvnw spring-boot:run
cd api-gateway && ./mvnw spring-boot:run
cd analytics-service && ./mvnw spring-boot:run
```

## Useful endpoints
- Auth (via gateway):
  - `POST http://localhost:4004/auth/login`
  - `GET http://localhost:4004/auth/validate`
- Patients (via gateway, JWT required):
  - `GET http://localhost:4004/api/patients`
  - `POST http://localhost:4004/api/patients`
  - `PUT http://localhost:4004/api/patients/{id}`
  - `DELETE http://localhost:4004/api/patients/{id}`
- OpenAPI (via gateway):
  - `http://localhost:4004/api-docs/auth`
  - `http://localhost:4004/api-docs/patients`

## Example requests
- HTTP: `api-requests/` (.http files)
- gRPC: `grpc-requests/billing-service/create-billing-account.http`

## Integration tests
The `integration-tests` module uses RestAssured and assumes the gateway is running at `http://localhost:4004`.
```bash
cd integration-tests && mvn test
```

## What I learned
From what I learned a microservice architecture helps scale applications by splitting multiples independent services.
Each services can be deployed and scaled based on its own need.  
In microservices architecture, an API Gateway is used as a single entry point to route requests to backend services.  
It can also help secure API route using filtering that validates JWT tokens.

For synchronous service-to-service communications gRPC uses protocol buffers. 
The contract of the request and the response is already specified on the .proto files.  
gRPC is faster and more efficient than REST for internal communication because it uses binary serialization and HTTP/2.
For client to server communication REST is still used ( I've heard about webRPC, I think it's a way to have faster client-server communication using protobuf..)

For asynchronous communications, for example in an analytics system that processes large numbers of events at the same time we can use a message broker like Apache Kafka.  
Events are produced by a producer and published into topics.
Consumers subscribe to these topics and we can also group them into a consumer group to process events in parallel. Kafka stores events durably in an ordered log within each partition using offset, allowing high throughput, scalability and fault tolerance.
  


