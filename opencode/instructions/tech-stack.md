# Technology Stack

## Purpose
This document defines the standard technology stack used across projects.
It helps agents understand project structure, make correct assumptions about conventions, and apply appropriate best practices during planning, implementation, and review.

## Scope
This applies to all projects unless explicitly overridden by project-specific configuration.
Agents should use this knowledge to:
- Recognize file structures and naming conventions.
- Apply framework-specific best practices.
- Generate correct build/run commands.
- Validate dependencies and configurations.

---

## Backend

### Java ecosystem
- **Frameworks**: Spring Boot, Quarkus
- **Build tool**: Maven (lifecycle management)
- **Data access**: JPA (primary), MyBatis (query-heavy scenarios)

### Key conventions
- Follow standard Maven project structure (`src/main/java`, `src/main/resources`, `src/test/java`).
- Spring Boot: use standard annotations (`@RestController`, `@Service`, `@Repository`, `@Component`).
- Quarkus: use CDI annotations and Quarkus extensions.
- JPA: entity classes in `domain` or `entity` packages; repositories follow naming pattern `*Repository`.
- MyBatis: mapper XML files in `src/main/resources/mappers`; mapper interfaces in dedicated package.

### Configuration files
- Spring Boot: `application.properties` or `application.yml` (profiles: `dev`, `test`, `prod`).
- Quarkus: `application.properties` (profiles via `%dev`, `%test`, `%prod` prefix).
- Maven: `pom.xml` for dependencies and build configuration.

---

## Frontend

### Framework
- **Angular**

### Key conventions
- Follow Angular CLI standard structure (`src/app`, `src/assets`, `src/environments`).
- Components: one component per file, use Angular naming conventions (`*.component.ts`, `*.component.html`, `*.component.css`).
- Services: use dependency injection, store in `services/` or feature-specific folders.
- Modules: feature modules and shared modules for reusability.
- Routing: use Angular Router with lazy loading where appropriate.

### Configuration files
- `angular.json`: workspace and project configuration.
- `package.json`: dependencies and scripts.
- `tsconfig.json`: TypeScript compiler options.

---

## Databases

### Supported databases
- **Oracle** (primary for enterprise/production)
- **PostgreSQL** (common for development/cloud deployments)

### Key conventions
- Use JPA entity mappings for standard CRUD.
- Use MyBatis for complex queries, reporting, or legacy schema integration.
- Database migrations: prefer Flyway or Liquibase (if used in project).
- Local development: databases run in Docker containers on Ubuntu Server VM.

### Connection configuration
- Spring Boot: configure in `application.properties` or `application.yml` (use profiles for dev/prod).
- Quarkus: configure in `application.properties` (use `%dev`/`%prod` profile prefixes).

---

## Build and dependency management

### Maven
- **Purpose**: Lifecycle management, dependency resolution, build automation.
- **Key files**: `pom.xml` (project object model).
- **Common commands**:
  - `mvn clean install`: clean build and install to local repo.
  - `mvn spring-boot:run` (Spring Boot): run application locally.
  - `mvn quarkus:dev` (Quarkus): run in dev mode with live reload.
  - `mvn test`: run unit tests.
  - `mvn package`: build deployable artifact (JAR/WAR).

### Dependency management
- Use Maven Central for public dependencies.
- Internal/company dependencies: use company artifact repository (if applicable).
- Version management: prefer properties in `pom.xml` for consistent versioning across modules.

---

## Local development environment

### Docker (on Ubuntu Server VM)
- **Purpose**: Run databases, services, and isolated environments locally.
- **Common use cases**:
  - PostgreSQL container for local development.
  - Oracle container (if available/licensed).
  - Other services (Redis, Kafka, etc.) as needed.

### Key conventions
- Use `docker-compose.yml` for multi-container setups.
- Mount volumes for persistent data (databases).
- Expose ports for local access from host machine.

---

## CI/CD and deployment

### Jenkins (client-side)
- **Purpose**: Continuous integration and deployment pipelines.
- **Key conventions**:
  - `Jenkinsfile`: declarative or scripted pipeline definition.
  - Stages: checkout, build (`mvn clean install`), test (`mvn test`), package, deploy.
  - Artifacts: JAR/WAR files produced by Maven.

### OpenShift (cloud deployment)
- **Purpose**: Container orchestration and cloud deployment.
- **Key conventions**:
  - Deployments use containerized applications (Docker images).
  - Configuration via `DeploymentConfig` or Kubernetes-native resources.
  - Environment-specific configs via ConfigMaps and Secrets.
  - Routes for external access.

### Deployment artifacts
- Spring Boot: fat JAR (executable JAR with embedded server).
- Quarkus: native executable or JVM-mode JAR.
- Frontend (Angular): static assets (build output from `ng build --prod`).

---

## Version control

### Git (personal projects)
- Use conventional commits format.
- Branch naming: `feat/<feature>`, `fix/<bug>`, `chore/<task>`.

### SVN (company projects)
- Follow company branching/tagging conventions.
- Use `trunk` for mainline development (if applicable).

---

## Agents affected

### Primary consumers of this knowledge
- **plan**: uses stack knowledge to analyze code structure and propose implementation plans.
- **build**: uses stack knowledge to implement code following framework conventions.
- **Code_Reviewer**: uses stack knowledge to validate code against framework best practices.
- **Repo_Manager**: uses stack knowledge to generate appropriate commit messages and commands.

### Secondary consumers
- **Task_Manager**: may reference stack when creating tasks with `paths_hint` or `dont_touch_paths`.

---

## Project-specific overrides
If a specific project deviates from this stack:
- Document exceptions in project-local `AGENTS.md` or `.opencode/` instructions.
- Provide explicit paths, dependencies, or commands that differ from defaults.

---

## Summary (quick reference)
- **Backend**: Java (Spring Boot/Quarkus), Maven, JPA/MyBatis
- **Frontend**: Angular
- **Databases**: Oracle, PostgreSQL
- **Local dev**: Docker on Ubuntu Server VM
- **CI/CD**: Jenkins pipelines
- **Deployment**: OpenShift (cloud), containerized apps
- **Version control**: Git (personal), SVN (company)
