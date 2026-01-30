# Technology Stack (Global Defaults)

## Purpose
Defines the default tech stack assumptions used by agents for analysis, planning, implementation, and review. Projects can override via project-local AGENTS.md.

## Scope
Applies unless explicitly overridden. Agents use this to recognize conventions and produce correct suggestions and review criteria.

---

## Backend

### Java ecosystem
- Frameworks: Spring Boot, Quarkus
- Build tool: Maven
- Data access: JPA (primary), MyBatis (query-heavy scenarios)

### Key conventions
- Maven structure: `src/main/java`, `src/main/resources`, `src/test/java`.
- Spring Boot annotations: `@RestController`, `@Service`, `@Repository`, `@Component`.
- Quarkus: CDI + Quarkus extensions.
- JPA: entities in `domain`/`entity`, repositories `*Repository`.
- MyBatis: mapper XML in `src/main/resources/mappers`, mapper interfaces in dedicated package.

### Configuration
- Spring Boot: `application.properties` or `application.yml` with profiles `dev/test/prod`.
- Quarkus: `application.properties` with `%dev/%test/%prod`.
- Maven: `pom.xml`.

---

## Frontend
- Framework: Angular
- Conventions: standard Angular CLI structure (`src/app`, `src/assets`, `src/environments`).

---

## Databases
- Oracle (enterprise/production default)
- PostgreSQL (common for dev/cloud)
- Migrations: prefer Flyway/Liquibase if used by the project.
- Local dev: databases in Docker on Ubuntu Server VM.

---

## Build & dependency management (reference)
Common Maven commands are listed as reference for humans. Agents must not execute builds/tests automatically unless explicitly instructed by the user.

Examples:
- `mvn clean install`, `mvn test`, `mvn package`, `mvn spring-boot:run`, `mvn quarkus:dev`

---

## CI/CD and deployment
- Jenkins pipelines (client-side), `Jenkinsfile`.
- OpenShift deployments, configuration via ConfigMaps/Secrets, routes for access.

---

## Version control
- Git for personal projects; conventional commits encouraged.
- SVN for company projects; follow company conventions.

---

## Agents affected
- plan/build: use these conventions for correct structure and choices.
- Code_Reviewer: validate changes against framework best practices.
- Repo_Manager: use context for commit messages (but derived from diff, not only tasks).
