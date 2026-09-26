# Scalable E-Commerce Platform

A microservices-based e-commerce backend built with Spring Boot and Spring Cloud. The platform is split into independently deployable services that register with a central discovery server and are exposed to clients through a single API gateway.

## Architecture

```
                ┌───────────────────┐
                │   API Gateway     │  (port 8080)
                │  Spring Cloud     │
                │  Gateway          │
                └─────────┬─────────┘
                          │ routes via Eureka
        ┌─────────────────┼─────────────────┐
        │                                   │
┌───────▼────────┐                 ┌────────▼────────┐
│  User Service   │                 │ Product Service  │
│  (port 8081)    │                 │  (port 8082)     │
└───────┬────────┘                 └────────┬────────┘
        │                                   │
        └─────────────┬─────────────────────┘
                       │ registers with
              ┌────────▼─────────┐
              │ Discovery Server  │  (port 8761)
              │  (Eureka)         │
              └───────────────────┘
```

## Services

| Service | Port | Description |
|---|---|---|
| **discovery-server** | 8761 | Eureka service registry that all other services register with |
| **api-gateway** | 8080 | Spring Cloud Gateway; routes external requests to the appropriate downstream service |
| **user-service** | 8081 | User registration, login, and JWT-based authentication (MySQL-backed) |
| **product-service** | 8082 | Product catalog CRUD operations (MySQL-backed) |

## Tech Stack

- Java 21
- Spring Boot 4.1.1 / Spring Cloud 2025.1.3
- Spring Cloud Gateway (WebFlux)
- Spring Cloud Netflix Eureka (service discovery)
- Spring Data JPA + MySQL
- Spring Security + JWT (`jjwt`)
- Bean Validation (`spring-boot-starter-validation`)
- Spring Boot Actuator

## Prerequisites

- JDK 21+
- Maven (or use the included `mvnw` wrapper)
- MySQL running locally, with the following databases created:
  - `user_db`
  - `product_db`

## Configuration

Both `user-service` and `product-service` connect to MySQL. Set the following environment variables before starting them (do **not** commit real credentials to `application.properties`):

| Variable | Used by | Description |
|---|---|---|
| `DB_USERNAME` | user-service | MySQL username (defaults to `root`) |
| `DB_PASSWORD` | user-service | MySQL password |
| `JWT_SECRET` | user-service | Secret key used to sign JWTs |
| `JWT_EXPIRATION` | user-service | Token expiry in ms (defaults to `3600000`) |

> **Note:** `product-service/src/main/resources/application.properties` currently has a MySQL username/password committed in plain text. Move these to environment variables (as `user-service` does) and rotate that password, since it's exposed in the repository's history.

## Running the Platform

Start the services **in this order** so that each one can register with Eureka and be discovered by the gateway:

1. **Discovery Server**
   ```bash
   cd discovery-server
   ./mvnw spring-boot:run
   ```
   Eureka dashboard: http://localhost:8761

2. **User Service**
   ```bash
   cd user-service
   export DB_PASSWORD=your_mysql_password
   export JWT_SECRET=your_jwt_secret
   ./mvnw spring-boot:run
   ```

3. **Product Service**
   ```bash
   cd product-service
   ./mvnw spring-boot:run
   ```

4. **API Gateway**
   ```bash
   cd api-gateway
   ./mvnw spring-boot:run
   ```
   Gateway entry point: http://localhost:8080

## API Endpoints (via the Gateway)

### User Service — `/api/users`
| Method | Path | Description |
|---|---|---|
| POST | `/api/users/register` | Register a new user |
| POST | `/api/users/login` | Log in and receive a JWT |
| GET | `/api/users` | List all users |
| GET | `/api/users/{id}` | Get a user by ID |

### Product Service — `/api/products`
| Method | Path | Description |
|---|---|---|
| POST | `/api/products` | Create a product |
| GET | `/api/products` | List all products |
| GET | `/api/products/{id}` | Get a product by ID |
| PUT | `/api/products/{id}` | Update a product |
| DELETE | `/api/products/{id}` | Delete a product |

## Health Checks

Each service exposes Actuator health/info endpoints, e.g.:
```
GET http://localhost:8081/actuator/health
GET http://localhost:8082/actuator/health
```

## Project Structure

```
Scalable-E-Commerce-Platform/
├── discovery-server/   # Eureka service registry
├── api-gateway/        # Spring Cloud Gateway routing layer
├── user-service/       # Auth & user management
└── product-service/    # Product catalog
```
