# Agent 5 — DevOps Engineer

## Role
The DevOps Engineer is the **fifth** agent in the pipeline. It handles everything needed to containerize, deploy, and document the microservice. It produces a production-ready `Dockerfile`, a `docker-compose.yml` for local development, and a `README.md` with setup and usage instructions.

---

## Input
- Full output from **Agent 1 (System Analyst)**: service name, tech stack (Java version, Spring Boot version, database type, build tool).
- Full output from **Agent 2 (Architect)**: package structure and base package name (used in the README).
- Full output from **Agent 3 (Database Agent)**: `application.properties` (to understand database connection details for docker-compose environment variables).

---

## What It Does

1. **Generates a multi-stage Dockerfile**:
   - **Stage 1 (build)**: Uses a JDK image (matching the specified Java version) to compile the project with Maven or Gradle and produce the executable JAR.
   - **Stage 2 (runtime)**: Uses a lightweight JRE image to run only the compiled JAR — minimizing the final image size and attack surface.
   - Includes `EXPOSE` for the correct port, `ENTRYPOINT` for running the JAR, and environment variable support.

2. **Generates `docker-compose.yml`**:
   - Defines the application service (built from the Dockerfile) and the database service (MySQL/PostgreSQL official image).
   - Sets up environment variables for database credentials and Spring datasource URL.
   - Configures `depends_on` to ensure the database starts before the application.
   - Defines a named Docker volume for database persistence.
   - Maps the correct ports (e.g. `8080:8080` for the app, `3306:3306` for MySQL).

3. **Generates `README.md`**:
   - Project overview and description
   - Prerequisites (Java, Docker, Maven/Gradle)
   - How to build and run the project locally (with and without Docker)
   - How to run with Docker Compose
   - API endpoint overview
   - Environment variable reference

---

## Output (Pydantic Schema: `DevOpsOutput`)

```json
{
  "dockerfile": "FROM maven:3.9-eclipse-temurin-17 AS build\n...\nFROM eclipse-temurin:17-jre\n...",
  "docker_compose": "version: '3.8'\nservices:\n  app:\n    build: .\n    ports:\n      - \"8080:8080\"\n  db:\n    image: mysql:8\n    ...",
  "readme": "# Order Service\n\n## Overview\n...\n## Running with Docker\n..."
}
```

---

## Multi-Stage Build Detail
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Role in the Pipeline
The DevOps Engineer is the second-to-last agent. Its output is standalone and does not feed into subsequent agents. However, it is critical for the **end user** who wants to actually run the generated service — without the Dockerfile and docker-compose, the service cannot easily be deployed or tested in isolation.

---

## Key Responsibilities Summary
| Responsibility | Description |
|---|---|
| Multi-stage Dockerfile | Optimized build + minimal runtime container |
| docker-compose.yml | Full local dev environment with app + database |
| README.md | Human-readable setup, build, and run instructions |

