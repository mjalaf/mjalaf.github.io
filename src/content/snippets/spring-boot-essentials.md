---
title: "Spring Boot Essentials"
description: "Day-to-day Spring Boot commands and patterns: project setup with Maven/Gradle, profiles, configuration, actuator, testing, and native images."
category: "Spring Boot"
---

## Project bootstrap

```bash
# Generate via Spring Initializr from the CLI
curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.4.0 \
  -d javaVersion=21 \
  -d groupId=com.mjalaf \
  -d artifactId=demo \
  -d name=demo \
  -d packageName=com.mjalaf.demo \
  -d dependencies=web,actuator,data-jpa,validation,security,lombok \
  -o demo.zip
unzip demo.zip -d demo && cd demo

# Or with the Spring Boot CLI (sdk install springboot)
spring init --build=maven --java-version=21 --boot-version=3.4.0 \
  --dependencies=web,actuator,data-jpa demo
```

## Run, build, watch

```bash
# Maven
./mvnw spring-boot:run
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
./mvnw clean package                    # produces target/*.jar
./mvnw clean package -DskipTests
java -jar target/demo-0.0.1-SNAPSHOT.jar

# Gradle
./gradlew bootRun
./gradlew bootRun --args='--spring.profiles.active=dev'
./gradlew clean bootJar
java -jar build/libs/demo-0.0.1-SNAPSHOT.jar

# Hot reload (devtools must be on classpath)
#   Maven:  <dependency>org.springframework.boot:spring-boot-devtools</dependency>
#   Gradle: developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

## Dependency management

```xml
<!-- Maven: add a starter -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

```bash
# Maven helpers
./mvnw dependency:tree
./mvnw versions:display-dependency-updates
./mvnw versions:display-plugin-updates

# Gradle helpers
./gradlew dependencies --configuration runtimeClasspath
./gradlew dependencyUpdates       # via com.github.ben-manes.versions plugin
```

## Profiles & configuration

```yaml
# application.yml — base config
spring:
  application:
    name: demo
  datasource:
    url: jdbc:postgresql://localhost:5432/demo
    username: demo
  jpa:
    hibernate:
      ddl-auto: validate
    open-in-view: false

server:
  port: 8080
  forward-headers-strategy: framework

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      probes:
        enabled: true                 # /actuator/health/liveness + /readiness
```

```yaml
# application-dev.yml — activated when SPRING_PROFILES_ACTIVE=dev
spring:
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update
logging:
  level:
    org.springframework.web: DEBUG
```

```bash
# Activate profile at runtime
java -jar app.jar --spring.profiles.active=prod
SPRING_PROFILES_ACTIVE=prod java -jar app.jar
```

> Property precedence (highest first): command-line args → `SPRING_APPLICATION_JSON` → OS env vars → `application-<profile>.yml` → `application.yml` → `@PropertySource` → defaults.

## Minimal REST controller

```java
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor                // Lombok
public class UserController {
    private final UserService service;

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> get(@PathVariable Long id) {
        return service.findById(id)
            .map(ResponseEntity::ok)
            .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserDto create(@Valid @RequestBody CreateUserRequest req) {
        return service.create(req);
    }
}
```

## Actuator — production observability

```bash
# Common endpoints (after enabling them in application.yml)
curl http://localhost:8080/actuator/health
curl http://localhost:8080/actuator/health/liveness
curl http://localhost:8080/actuator/health/readiness
curl http://localhost:8080/actuator/info
curl http://localhost:8080/actuator/metrics
curl http://localhost:8080/actuator/metrics/jvm.memory.used
curl http://localhost:8080/actuator/prometheus   # scrape endpoint
curl http://localhost:8080/actuator/env
curl http://localhost:8080/actuator/loggers
```

```bash
# Change log level at runtime (no restart)
curl -X POST -H "Content-Type: application/json" \
  -d '{"configuredLevel":"DEBUG"}' \
  http://localhost:8080/actuator/loggers/com.mjalaf.demo
```

## Testing

```bash
./mvnw test
./mvnw test -Dtest=UserControllerTest
./mvnw verify                            # runs integration tests in failsafe
./gradlew test --tests "*UserControllerTest"
```

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Testcontainers
class UserApiIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        r.add("spring.datasource.url",      postgres::getJdbcUrl);
        r.add("spring.datasource.username", postgres::getUsername);
        r.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired MockMvc mvc;

    @Test
    void healthIsUp() throws Exception {
        mvc.perform(get("/actuator/health"))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.status").value("UP"));
    }
}
```

## Docker & containers

```bash
# Spring Boot can build an OCI image with no Dockerfile (Cloud Native Buildpacks)
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=mjalaf/demo:0.0.1
./gradlew bootBuildImage --imageName=mjalaf/demo:0.0.1

docker run -p 8080:8080 mjalaf/demo:0.0.1
```

```dockerfile
# Multi-stage Dockerfile (classic approach)
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app
COPY . .
RUN ./mvnw -q -DskipTests package

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

## GraalVM native image (Spring Boot 3+)

```bash
# Requires GraalVM 21+. Add the native-maven-plugin (auto-configured by Spring Initializr).
./mvnw -Pnative native:compile
./target/demo                        # ~50 MB binary, ~50 ms startup

# Native container image
./mvnw spring-boot:build-image -Pnative
```

## Database migrations

```bash
# Flyway (recommended for SQL)
src/main/resources/db/migration/V1__init.sql
src/main/resources/db/migration/V2__add_users.sql

./mvnw flyway:migrate
./mvnw flyway:info
./mvnw flyway:repair

# Liquibase
src/main/resources/db/changelog/db.changelog-master.yaml
./mvnw liquibase:update
./mvnw liquibase:rollback -Dliquibase.rollbackCount=1
```

## Useful command-line overrides

```bash
# Bind to all interfaces, change port, set datasource via env
SERVER_PORT=9090 \
SERVER_ADDRESS=0.0.0.0 \
SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/demo \
java -jar app.jar

# Increase log verbosity for one package
java -jar app.jar --logging.level.org.hibernate.SQL=DEBUG

# Generate a thread dump on a running JVM
jcmd <PID> Thread.print > thread-dump.txt

# Heap dump on OutOfMemoryError
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heap.hprof -jar app.jar
```

## Spring Boot versions at a glance

| Version | Released  | Java baseline | Notes |
| ------- | --------- | ------------- | ----- |
| 2.7.x   | May 2022  | Java 8        | Last 2.x line (EOL community; commercial support via Broadcom/Tanzu) |
| 3.0.x   | Nov 2022  | **Java 17**   | Jakarta EE 10 namespace (`javax.*` → `jakarta.*`), GraalVM native GA |
| 3.1.x   | May 2023  | Java 17       | Docker Compose support, Testcontainers @ServiceConnection |
| 3.2.x   | Nov 2023  | Java 17       | Virtual threads (Loom) support, RestClient |
| 3.3.x   | May 2024  | Java 17       | CDS support, structured logging |
| 3.4.x   | Nov 2024  | Java 17       | Current GA; improved native image + observability |

> Migrating from 2.x to 3.x: switch to Java 17+, rename `javax.*` → `jakarta.*`, and re-validate dependencies. Use [Spring Boot Migrator](https://github.com/spring-projects-experimental/spring-boot-migrator) or OpenRewrite recipes.
