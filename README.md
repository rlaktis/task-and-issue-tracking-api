# TaskEngine: Task & Issue Tracking REST API

REST API for managing tasks, built with **Java 21**, **Spring Boot 3.3**, a layered architecture, **Jakarta Bean Validation** and **RFC 7807 `ProblemDetail`** error responses. Tasks are kept in memory, so there is no database setup.

---

## Features

- **Layered architecture** (`controller` → `service` → `model` / `dto`): each layer has its own responsibility. Dependencies are injected through constructors.

- **DTOs as Java records** (`dto/`): `CreateTaskRequest`, `UpdateTaskRequest` and `TaskResponse` keep the domain model separate from the API. `TaskResponse.from(Task)` maps a task to a response.

- **In-memory storage** (`service/TaskService.java`): tasks are stored in a `ConcurrentHashMap<UUID, Task>`. `Task` is an immutable record, so an update creates a new `Task` that keeps the original `id` and `createdAt`. Data is lost when the application stops.

- **Validation** (Jakarta Bean Validation): `@NotBlank`, `@Size(max = 100)` and `@NotNull` on request payloads, so blank strings and missing enum values are rejected before reaching the business logic.

- **Error handling** (`exception/GlobalExceptionHandler.java`): a `@RestControllerAdvice` returns RFC 7807 `ProblemDetail` responses for `404` and `400`. Validation errors include a map of the failed fields under `errors`.

- **Tests** (16 in total):
  - **Unit tests** (`TaskServiceTest`, 8): service logic with JUnit 5 and AssertJ.
  - **Controller tests** (`TaskControllerTest`, 7): `@WebMvcTest` with MockMvc, covering routing, JSON payloads, headers and RFC 7807 error responses.
  - **Context test** (`TaskEngineApplicationTests`): checks that the application context starts.

---

## Tech Stack

- **Language:** [Java 21](https://openjdk.org/projects/jdk/21/)
- **Framework:** [Spring Boot 3.3.5](https://spring.io/projects/spring-boot)
- **Build tool:** [Maven Wrapper (mvnw)](https://maven.apache.org/)
- **Validation:** [Jakarta Bean Validation](https://beanvalidation.org/) (Hibernate Validator)
- **Testing:** [JUnit 5](https://junit.org/junit5/), [AssertJ](https://assertj.github.io/doc/), [MockMvc](https://docs.spring.io/spring-framework/reference/testing/spring-mvc-test-framework.html), [Mockito](https://site.mockito.org/)
- **Error format:** [RFC 7807 Problem Details](https://datatracker.ietf.org/doc/html/rfc7807)

---

## Project Structure

```text
src/
├── main/
│   └── java/com/devpulse/taskengine/
│       ├── TaskEngineApplication.java            # Application entry point
│       ├── controller/
│       │   └── TaskController.java               # REST controller (5 endpoints)
│       ├── dto/
│       │   ├── CreateTaskRequest.java            # Request record for POST
│       │   ├── UpdateTaskRequest.java            # Request record for PUT
│       │   └── TaskResponse.java                 # Response record with static mapper
│       ├── exception/
│       │   ├── GlobalExceptionHandler.java       # @RestControllerAdvice, ProblemDetail responses
│       │   └── TaskNotFoundException.java        # Unchecked exception, maps to 404
│       ├── model/
│       │   ├── Priority.java                     # Enum (LOW, MEDIUM, HIGH)
│       │   ├── Task.java                         # Immutable record (UUID, Instant)
│       │   └── TaskStatus.java                   # Enum (TODO, IN_PROGRESS, COMPLETED)
│       └── service/
│           └── TaskService.java                  # Business logic and in-memory store
└── test/
    └── java/com/devpulse/taskengine/
        ├── TaskEngineApplicationTests.java       # Application context test
        ├── controller/
        │   └── TaskControllerTest.java           # MockMvc web slice tests
        └── service/
            └── TaskServiceTest.java              # Unit tests (JUnit 5, AssertJ)
```

---

## API Endpoints

Base URL: `http://localhost:8080/api/tasks`

| Method | Endpoint | Description | Status | Response |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/api/tasks` | Create a task | `201 Created` | `TaskResponse` + `Location` header |
| `GET` | `/api/tasks` | List all tasks | `200 OK` | `List<TaskResponse>` |
| `GET` | `/api/tasks/{id}` | Get a task by UUID | `200 OK` | `TaskResponse` (or `404 ProblemDetail`) |
| `PUT` | `/api/tasks/{id}` | Update a task | `200 OK` | `TaskResponse` (or `404 ProblemDetail`) |
| `DELETE` | `/api/tasks/{id}` | Delete a task | `204 No Content` | Empty body (or `404 ProblemDetail`) |

---

## Getting Started

### Prerequisites
- Java JDK 21 or newer
- Git

### Running the project

1. Clone the repository:
   ```bash
   git clone https://github.com/rlaktis/task-and-issue-tracking-api.git
   cd task-and-issue-tracking-api
   ```

2. Run the tests:
   ```bash
   # Windows PowerShell
   .\mvnw.cmd test

   # macOS / Linux
   ./mvnw test
   ```

3. Start the application:
   ```bash
   # Windows PowerShell
   .\mvnw.cmd spring-boot:run

   # macOS / Linux
   ./mvnw spring-boot:run
   ```

4. The API is then available at:
   ```text
   http://localhost:8080/api/tasks
   ```

5. Optionally, build an executable JAR:
   ```bash
   # Windows PowerShell
   .\mvnw.cmd clean package
   java -jar target/task-engine-0.0.1-SNAPSHOT.jar
   ```

---

## Limitations and Next Steps

- Tasks are stored in memory only, so data is lost on restart. A database (for example PostgreSQL with Spring Data JPA) would be the next step.
- No authentication or authorization.
- `GET /api/tasks` returns all tasks, with no pagination or filtering.
