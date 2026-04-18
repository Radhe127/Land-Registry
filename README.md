# 🏠 Land Registry

A production-ready backend system for managing land records, ownership verification, and secure ownership transfers. Built entirely using Spring Boot, this application leverages PostgreSQL and Hibernate (JPA) to deliver a scalable, secure, and maintainable architecture.

---

## 🗂️ Table of Contents

- [Overview](#overview)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Domain Model](#domain-model)
- [API Endpoints](#api-endpoints)
- [Security](#security)
- [Validation & Error Handling](#validation--error-handling)
- [Configuration](#configuration)
- [Running the Application](#running-the-application)
- [Build & Deployment](#build--deployment)
- [Testing Strategy](#testing-strategy)
- [Future Enhancements](#future-enhancements)

---

## 🏘️ Overview

The Land Registry backend is designed to support:

- 🏡 Digital land registration
- 🏠 Ownership verification (Admin/Officer controlled)
- 🏘️ Secure ownership transfer
- 🧾 Immutable ownership history tracking
- 🔐 Role-based access control (RBAC)

---

## 🧰 Technology Stack

### Core Backend
- **Java 17+** (Java 21 recommended)
- **Spring Boot 3+**

### Frameworks & Libraries
- Spring Web (REST APIs)
- Spring Data JPA (Hibernate ORM)
- Spring Security (JWT Authentication)
- Jakarta Bean Validation
- Lombok (optional)

### Database
- PostgreSQL

### Tools
- Maven
- Docker (optional)
- Swagger/OpenAPI (API documentation)
- Postman (API testing)

---

## 🏛️ Architecture

The backend follows a layered architecture pattern:

Controller → Service → Repository → Database

---


### Request Lifecycle

1. Client sends request with JWT token
2. Spring Security validates token and roles
3. Controller handles HTTP request
4. Service processes business logic
5. Repository interacts with database via JPA
6. JSON response returned to client

---

## 🗃️ Project Structure
```text
backend/
├── src/main/java/com/landregistry/
│   ├── config/        # App & security configuration
│   ├── controller/    # REST controllers
│   ├── dto/           # Data transfer objects
│   ├── entity/        # JPA entities
│   ├── repository/    # Database access layer
│   ├── security/      # JWT & authentication
│   ├── service/       # Business logic
│   └── exception/     # Global error handling
├── src/main/resources/
│   └── application.yml
└── pom.xml
```

This layout is the intended backend structure for the application.


---

## 🧬 Domain Model

### 👤 User Entity

| Field          | Type      | Description                   |
|----------------|-----------|-------------------------------|
| id             | Long      | Primary key                   |
| fullName       | String    | User's full name              |
| email          | String    | Unique, used for login        |
| passwordHash   | String    | BCrypt encoded                |
| role           | Enum      | ADMIN / OFFICER / USER        |
| createdAt      | Timestamp | Account creation timestamp    |

### 🌍 Land Entity

| Field          | Type      | Description                          |
|----------------|-----------|--------------------------------------|
| id             | Long      | Primary key                          |
| surveyNumber   | String    | Unique land identifier               |
| area           | Double    | Area in hectares / sq meters         |
| location       | String    | Address / coordinates                |
| ownerId        | Long      | Reference to User (current owner)    |
| marketValue    | Double    | Estimated monetary value             |
| status         | Enum      | PENDING_VERIFICATION / VERIFIED / REJECTED |
| createdAt      | Timestamp | Registration timestamp               |

### 🔁 LandTransfer Entity

| Field          | Type      | Description                    |
|----------------|-----------|--------------------------------|
| id             | Long      | Primary key                    |
| landId         | Long      | Reference to Land              |
| fromOwnerId    | Long      | Previous owner                 |
| toOwnerId      | Long      | New owner                      |
| transferDate   | Timestamp | Date of transfer               |
| remarks        | String    | Optional notes                 |

### 🏷️ Enums

**Roles:**
- `ROLE_ADMIN`
- `ROLE_OFFICER`
- `ROLE_USER`

**Land Status:**
- `PENDING_VERIFICATION`
- `VERIFIED`
- `REJECTED`

---

## 📡 API Endpoints

### 🔑 Authentication

| Method | Endpoint               | Description         |
|--------|------------------------|---------------------|
| POST   | `/api/auth/register`   | Register new user   |
| POST   | `/api/auth/login`      | Login & get JWT     |

### 🏠 Land Management

| Method | Endpoint                 | Description                      | Access        |
|--------|--------------------------|----------------------------------|---------------|
| POST   | `/api/lands`             | Register new land                | Authenticated |
| GET    | `/api/lands`             | List all lands                   | Authenticated |
| GET    | `/api/lands/{id}`        | Get land details                 | Authenticated |
| GET    | `/api/lands/search`      | Search land (by survey number)   | Authenticated |
| PUT    | `/api/lands/{id}`        | Update land info                 | Owner/Admin   |
| DELETE | `/api/lands/{id}`        | Delete land                      | Admin only    |

### ✅ Verification (Admin/Officer only)

| Method | Endpoint                 | Description      |
|--------|--------------------------|------------------|
| PUT    | `/api/lands/{id}/verify` | Verify a land    |
| PUT    | `/api/lands/{id}/reject` | Reject a land    |

### 🔄 Transfer

| Method | Endpoint                    | Description                        | Access        |
|--------|-----------------------------|------------------------------------|---------------|
| POST   | `/api/lands/{id}/transfer`  | Transfer ownership to another user | Owner/Admin   |
| GET    | `/api/lands/{id}/history`   | Get transfer history for a land    | Authenticated |

---

## 🛡️ Security

### Authentication
- **JWT-based stateless authentication**
- Token format: `Authorization: Bearer <token>`

### Password Security
- BCrypt hashing for passwords

### Authorization
- Role-based access control (RBAC)
- Method-level security with `@PreAuthorize`

### Access Rules

| Endpoint                     | Access                     |
|------------------------------|----------------------------|
| `/api/auth/**`               | Public                     |
| `/api/lands/**`              | Authenticated              |
| `/api/lands/{id}/verify`     | Admin / Officer only       |
| `/api/lands/{id}/reject`     | Admin / Officer only       |
| `/api/lands/{id}` (DELETE)   | Admin only                 |

---

## ✅ Validation & Error Handling

### Validation Annotations
- `@NotNull`, `@Email`, `@Size`, `@Min`, `@Max`

### Error Handling
- Global exception handler (`@ControllerAdvice`)
- Standardized API error responses with proper HTTP status codes

---

## 🛠️ Configuration

### `application.yml`

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/land_registry
    username: postgres
    password: postgres

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

jwt:
  secret: your-secure-secret
  expiration-ms: 86400000   # 24 hours
```

> Note: Change the JWT secret and database credentials for production environments.

---

## 🚀 Running the Application

### 📋 Prerequisites
- Java 17+ installed
- Maven 3.9+ (or use the included Maven wrapper)
- PostgreSQL database running

### 🪜 Steps
1. Clone the repository.

```bash
git clone https://github.com/your-repo/land-registry-backend.git
cd land-registry-backend
```

2. Configure database.

Update `application.yml` with your PostgreSQL credentials.

3. Run with Maven.

```bash
./mvnw spring-boot:run
```

4. Access the API.

The server starts at `http://localhost:8080`.

---

## 📦 Build and Deployment

### 🧱 Build Executable JAR

```bash
./mvnw clean package
```

### ▶️ Run the JAR

```bash
java -jar target/land-registry-0.0.1-SNAPSHOT.jar
```

### ☁️ Deployment Options
- Docker: containerize using a `Dockerfile`
- Cloud platforms: AWS (EC2, ECS), Azure, GCP
- PaaS: Render, Railway, Heroku

---

## 🧪 Testing Strategy

| Test Type | Tools / Approach |
|-----------|------------------|
| Unit Tests | JUnit 5 + Mockito (service layer) |
| Integration Tests | `@SpringBootTest` + Testcontainers |
| Security Tests | JWT and role-based access tests |

Run tests with:

```bash
./mvnw test
```

---

## 🌟 Future Enhancements

- Document upload for land proof (images, PDFs)
- GIS-based land visualization
- Digital signature workflow for transfers
- Notification system (Email/SMS) for status changes
- Admin analytics dashboard with charts
