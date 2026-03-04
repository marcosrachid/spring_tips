## Spring & Spring Boot – Concise Guide to Key Concepts

English version. For the Portuguese version, see `README.pt-BR.md`.

This repository gathers **notes and practical examples** about the main concepts of **Spring Framework** and **Spring Boot**: core modules, commonly used libraries, lifecycle, configuration, async execution, testing, and best practices.

---

## 1. Overview

- **Spring Framework**: Java development platform focused on:
  - **Inversion of Control (IoC)** and **Dependency Injection (DI)**
  - **Aspect-Oriented Programming (AOP)**
  - Abstractions for **data**, **transactions**, **messaging**, **security**, etc.
- **Spring Boot**: a layer on top of Spring that:
  - Simplifies application **bootstrap** (very little manual configuration)
  - Uses **auto‑configuration** and **convention over configuration**
  - Embeds a server (Tomcat/Jetty/Undertow) and packages everything into a **fat jar**

---

## 2. Spring Core (Core / Context / Beans)

- **IoC Container**:
  - Manages the lifecycle of **beans** (objects managed by Spring).
  - Configuration via:
    - Annotations (`@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`…)
    - Java config (`@Bean` methods in `@Configuration` classes)
    - (Legacy) XML.
- **Most common scopes**:
  - `singleton` (default) – one instance per `ApplicationContext`
  - `prototype` – new instance at each injection
  - Web: `request`, `session`, `application`, `websocket`
- **Dependency injection**:
  - `@Autowired`, constructors, setters
  - Best practice: prefer **constructor injection**.

---

## 3. Spring Boot – Basic Structure

- **Main class**:
  - `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
  - `main` method with `SpringApplication.run(...)`.
- **Auto-configuration**:
  - Based on **classpath + properties** (e.g., if `spring-boot-starter-web` is present, it configures MVC + Tomcat).
  - Can be adjusted with `spring.autoconfigure.exclude` or specific annotations.
- **Profiles**:
  - `@Profile("dev")`, `@Profile("prod")`, etc.
  - Files: `application-dev.yml`, `application-prod.yml`.

---

## 4. Main Starters / Libraries

- **Web / APIs**:
  - `spring-boot-starter-web` – MVC, REST, Jackson, Tomcat.
  - `spring-boot-starter-webflux` – reactive programming (Netty).
- **Data**:
  - `spring-boot-starter-data-jpa` – JPA/Hibernate, repositories.
  - `spring-boot-starter-jdbc` – traditional JDBC with `JdbcTemplate`.
  - `spring-boot-starter-data-mongodb`, `data-redis`, etc.
- **Security**:
  - `spring-boot-starter-security` – authentication, authorization, filters, CSRF.
- **Messaging**:
  - `spring-boot-starter-amqp` (RabbitMQ), `spring-kafka`, `spring-cloud-stream`.
- **Observability**:
  - `spring-boot-starter-actuator` – metrics, health checks, info, etc.
- **Template engines / views**:
  - `spring-boot-starter-thymeleaf`, `spring-boot-starter-mustache`.
- **Testing**:
  - `spring-boot-starter-test` – JUnit, Mockito/MockK, AssertJ, Spring Test, etc.

---

## 5. Configuration: Properties / YAML / Profiles

- Main file:
  - `application.properties` or `application.yml`.
- Configuration sources (simplified order of precedence):
  - Environment variables
  - Command-line arguments
  - External files
  - Packaged file (`application.yml` inside the jar)
- **Property binding**:
  - `@Value("${my.config}")`
  - `@ConfigurationProperties(prefix = "app")` + POJO class (typed binding, recommended).

---

## 6. Web: Spring MVC & Controllers

- **REST Controller**:
  - `@RestController` + `@RequestMapping` (or `@GetMapping`, `@PostMapping`…)
- **Validation**:
  - `@Valid` + Bean Validation (`@NotNull`, `@Size`…)
  - Custom messages via `messages.properties`.
- **Error handling**:
  - `@ControllerAdvice` + `@ExceptionHandler`
  - Global exception handling and standardized responses (e.g., `ErrorResponse`).
- **JSON serialization**:
  - `Jackson` by default (Web MVC).
  - Customization via `ObjectMapper` or `Jackson2ObjectMapperBuilder`.

---

## 7. Data Access (JPA, Transactions, etc.)

- **Spring Data JPA**:
  - `Repository` / `CrudRepository` / `JpaRepository` interfaces.
  - Query methods (`findByEmail`, `findByStatusAndType`…).
  - `@Query` for JPQL / native SQL.
- **Transactions**:
  - `@Transactional` on service or repository methods/classes.
  - Configuration of:
    - Propagation (`REQUIRED`, `REQUIRES_NEW`…)
    - Isolation (`READ_COMMITTED`, `REPEATABLE_READ`…)
    - Timeouts and read-only.
- **Migrations**:
  - Integration with `Flyway` or `Liquibase` for schema versioning.

---

## 8. Security with Spring Security (Quick View)

- Filter-based pipelines:
  - Authentication, authorization, CSRF, security headers, etc.
- Common authentication methods:
  - **Form login**, **HTTP Basic**, **JWT**, **OAuth2/OpenID Connect**.
- Modern configuration (Spring Security 5.7+):
  - `SecurityFilterChain` beans instead of extending `WebSecurityConfigurerAdapter`.
- Useful annotations:
  - `@EnableMethodSecurity` / `@PreAuthorize`, `@PostAuthorize`.

---

## 9. Asynchronous Execution and Concurrency

- **Enable async**:
  - `@EnableAsync` on a configuration class or main app class.
- **Async methods**:
  - `@Async` on methods of Spring-managed beans.
  - Common return types:
    - `void`
    - `Future<T>`, `CompletableFuture<T>`
    - In reactive contexts: `Mono<T>`, `Flux<T>` (WebFlux/Reactor).
- **Executors (`TaskExecutor`)**:
  - Configure a named `ThreadPoolTaskExecutor` and reference it in `@Async("executorName")`.
  - Important for:
    - Controlling number of threads
    - Queue size
    - Rejection policy, etc.
- **Other async / scheduled mechanisms**:
  - `@Scheduled` for periodic tasks (`fixedRate`, `fixedDelay`, `cron`).
  - Messaging integration (Kafka, RabbitMQ) for background processing.

---

## 10. Application / Bean Lifecycle (Step by Step)

From `main` until the application is ready to serve requests:

- **1. `main` and `SpringApplication.run`**
  - You call:
    ```java
    @SpringBootApplication
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```
  - `SpringApplication` analyzes the main class, determines the app type (servlet / reactive / non-web) and registers default `ApplicationListener`s.

- **2. Environment preparation**
  - Builds the `Environment`:
    - Loads properties from command-line args, environment variables, `application.yml/properties`, external files, etc.
  - Publishes early events such as:
    - `ApplicationStartingEvent`
    - `ApplicationEnvironmentPreparedEvent`
  - Shows the banner if enabled.

- **3. Creation of the `ApplicationContext`**
  - Chooses context implementation:
    - Servlet: `AnnotationConfigServletWebServerApplicationContext`
    - Reactive: `AnnotationConfigReactiveWebServerApplicationContext`
    - Non-web: `AnnotationConfigApplicationContext`
  - Instantiates the context and wires:
    - `Environment`
    - `ApplicationListeners`
    - `ResourceLoader`, etc.

- **4. Registration of configuration classes and auto-configurations**
  - Registers:
    - Your main class (`@SpringBootApplication`) as `@Configuration`.
    - Other configs found via **component scan**.
  - Loads **auto-configurations**:
    - Discovers `@AutoConfiguration` classes via `spring.factories` / `AutoConfiguration.imports`.
    - Evaluates conditions (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.).
    - Only registers auto-configurations whose conditions are satisfied.

- **5. Context `refresh()` – core of the lifecycle**
  - Prepares the `BeanFactory`:
    - Registers default `BeanPostProcessor`s and `BeanFactoryPostProcessor`s.
  - Invokes `BeanFactoryPostProcessor`s:
    - E.g. configuration properties binding, customizations from Boot, etc.
  - Registers custom `BeanPostProcessor`s (provided by you or by auto-config).
  - Initializes message source (i18n) and application event multicaster.
  - **Creates the embedded server** (for web apps):
    - Creates a `ServletWebServerFactory` (Tomcat/Jetty/Undertow) and prepares the `WebServer`.
  - **Instantiates singleton beans**:
    - Calls constructors / dependency injection.
    - Executes `@PostConstruct`, `InitializingBean.afterPropertiesSet`, custom `initMethod`.
    - Applies `BeanPostProcessor` (`postProcessBeforeInitialization` / `postProcessAfterInitialization`).
  - Publishes `ContextRefreshedEvent`.

- **6. Embedded server “online” and startup events**
  - Starts the embedded server and binds the listening port (typically 8080).
  - Publishes:
    - `ApplicationStartedEvent` (context refreshed, before runners).
  - Executes:
    - `ApplicationRunner` and `CommandLineRunner` beans (in order, if `@Order` is used).
  - Publishes:
    - `ApplicationReadyEvent` (context fully initialized, server running, runners executed).

- **7. Steady state and shutdown**
  - From this point:
    - The app serves HTTP requests, processes messages, runs `@Scheduled` tasks, etc.
  - On shutdown:
    - Publishes `ContextClosedEvent`.
    - Calls `@PreDestroy`, `DisposableBean.destroy`, `SmartLifecycle.stop`, etc.

---

## 11. Observability: Actuator, Logs, Metrics

- **Spring Boot Actuator**:
  - Endpoints such as: `/actuator/health`, `/actuator/info`, `/actuator/metrics`, `/actuator/env`, etc.
  - Can integrate with **Micrometer** for Prometheus, Grafana, etc.
- **Logging**:
  - `spring-boot-starter-logging` (Logback by default).
  - Configuration via `application.yml` or `logback-spring.xml`.
- **Tracing / distributed tracing** (modern stack):
  - Spring Observability + Micrometer Tracing, integration with Zipkin/Jaeger/OpenTelemetry.

---

## 12. Testing with Spring Boot

- Unit tests:
  - JUnit + Mockito/MockK, without loading the Spring context (fast).
- Integration tests:
  - `@SpringBootTest` – loads the full application context.
  - `@WebMvcTest` – focuses on the web layer (controllers, filters).
  - `@DataJpaTest` – focuses on JPA repositories (usually with in-memory DB).
- Helper tools:
  - `TestRestTemplate` / `WebTestClient` for endpoint testing.
  - `@Sql` / `@SqlGroup` to prepare data.

---

## 13. Best Practices (Summary)

- **Layered architecture**:
  - Controller → Service → Repository → Domain
  - Avoid business logic in controllers or repositories.
- **Typed configuration**:
  - Prefer `@ConfigurationProperties` over scattered `@Value` usage.
- **Keep beans cohesive**:
  - Small, focused classes with a single responsibility.
- **Centralized exception handling**:
  - `@ControllerAdvice` for REST APIs.
- **Be careful with transactions and lazy loading**:
  - Avoid lazy access outside a transaction (`LazyInitializationException`).
  - Introduce DTOs when appropriate.
- **Async done responsibly**:
  - Configure an appropriate `TaskExecutor`.
  - Avoid blocking threads in reactive or async code.

---

## 14. Ideas for Examples in this Repository

- Simple REST API with:
  - Controllers, Services, Repositories (JPA)
  - Validation, global error handling
  - Basic security (JWT or Basic Auth)
- Asynchronous executions:
  - `@Async` methods with `CompletableFuture`
  - `@Scheduled` jobs
- Observability:
  - Actuator + basic metrics
- Tests:
  - `@WebMvcTest` for controllers
  - `@DataJpaTest` for repositories
  - `@SpringBootTest` for end-to-end flows

---

**Goal**: this `README` serves as a **mental map** of the Spring/Spring Boot ecosystem. As you add examples to this repository, try to reference them in the sections above to build a coherent and concise study material.

## Spring & Spring Boot – Concise Guide to Key Concepts

English version. For the Portuguese version, see `README.pt-BR.md`.

This repository gathers **notes and practical examples** about the main concepts of **Spring Framework** and **Spring Boot**: core modules, commonly used libraries, lifecycle, configuration, async execution, testing, and best practices.

---

## 1. Overview

- **Spring Framework**: Java development platform focused on:
  - **Inversion of Control (IoC)** and **Dependency Injection (DI)**
  - **Aspect-Oriented Programming (AOP)**
  - Abstractions for **data**, **transactions**, **messaging**, **security**, etc.
- **Spring Boot**: a layer on top of Spring that:
  - Simplifies application **bootstrap** (very little manual configuration)
  - Uses **auto‑configuration** and **convention over configuration**
  - Embeds a server (Tomcat/Jetty/Undertow) and packages everything into a **fat jar**

---

## 2. Spring Core (Core / Context / Beans)

- **IoC Container**:
  - Manages the lifecycle of **beans** (objects managed by Spring).
  - Configuration via:
    - Annotations (`@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`…)
    - Java config (`@Bean` methods in `@Configuration` classes)
    - (Legacy) XML.
- **Most common scopes**:
  - `singleton` (default) – one instance per `ApplicationContext`
  - `prototype` – new instance at each injection
  - Web: `request`, `session`, `application`, `websocket`
- **Dependency injection**:
  - `@Autowired`, constructors, setters
  - Best practice: prefer **constructor injection**.

---

## 3. Spring Boot – Basic Structure

- **Main class**:
  - `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
  - `main` method with `SpringApplication.run(...)`.
- **Auto-configuration**:
  - Based on **classpath + properties** (e.g., if `spring-boot-starter-web` is present, it configures MVC + Tomcat).
  - Can be adjusted with `spring.autoconfigure.exclude` or specific annotations.
- **Profiles**:
  - `@Profile("dev")`, `@Profile("prod")`, etc.
  - Files: `application-dev.yml`, `application-prod.yml`.

---

## 4. Main Starters / Libraries

- **Web / APIs**:
  - `spring-boot-starter-web` – MVC, REST, Jackson, Tomcat.
  - `spring-boot-starter-webflux` – reactive programming (Netty).
- **Data**:
  - `spring-boot-starter-data-jpa` – JPA/Hibernate, repositories.
  - `spring-boot-starter-jdbc` – traditional JDBC with `JdbcTemplate`.
  - `spring-boot-starter-data-mongodb`, `data-redis`, etc.
- **Security**:
  - `spring-boot-starter-security` – authentication, authorization, filters, CSRF.
- **Messaging**:
  - `spring-boot-starter-amqp` (RabbitMQ), `spring-kafka`, `spring-cloud-stream`.
- **Observability**:
  - `spring-boot-starter-actuator` – metrics, health checks, info, etc.
- **Template engines / views**:
  - `spring-boot-starter-thymeleaf`, `spring-boot-starter-mustache`.
- **Testing**:
  - `spring-boot-starter-test` – JUnit, Mockito/MockK, AssertJ, Spring Test, etc.

---

## 5. Configuration: Properties / YAML / Profiles

- Main file:
  - `application.properties` or `application.yml`.
- Configuration sources (simplified order of precedence):
  - Environment variables
  - Command-line arguments
  - External files
  - Packaged file (`application.yml` inside the jar)
- **Property binding**:
  - `@Value("${my.config}")`
  - `@ConfigurationProperties(prefix = "app")` + POJO class (typed binding, recommended).

---

## 6. Web: Spring MVC & Controllers

- **REST Controller**:
  - `@RestController` + `@RequestMapping` (or `@GetMapping`, `@PostMapping`…)
- **Validation**:
  - `@Valid` + Bean Validation (`@NotNull`, `@Size`…)
  - Custom messages via `messages.properties`.
- **Error handling**:
  - `@ControllerAdvice` + `@ExceptionHandler`
  - Global exception handling and standardized responses (e.g., `ErrorResponse`).
- **JSON serialization**:
  - `Jackson` by default (Web MVC).
  - Customization via `ObjectMapper` or `Jackson2ObjectMapperBuilder`.

---

## 7. Data Access (JPA, Transactions, etc.)

- **Spring Data JPA**:
  - `Repository` / `CrudRepository` / `JpaRepository` interfaces.
  - Query methods (`findByEmail`, `findByStatusAndType`…).
  - `@Query` for JPQL / native SQL.
- **Transactions**:
  - `@Transactional` on service or repository methods/classes.
  - Configuration of:
    - Propagation (`REQUIRED`, `REQUIRES_NEW`…)
    - Isolation (`READ_COMMITTED`, `REPEATABLE_READ`…)
    - Timeouts and read-only.
- **Migrations**:
  - Integration with `Flyway` or `Liquibase` for schema versioning.

---

## 8. Security with Spring Security (Quick View)

- Filter-based pipelines:
  - Authentication, authorization, CSRF, security headers, etc.
- Common authentication methods:
  - **Form login**, **HTTP Basic**, **JWT**, **OAuth2/OpenID Connect**.
- Modern configuration (Spring Security 5.7+):
  - `SecurityFilterChain` beans instead of extending `WebSecurityConfigurerAdapter`.
- Useful annotations:
  - `@EnableMethodSecurity` / `@PreAuthorize`, `@PostAuthorize`.

---

## 9. Asynchronous Execution and Concurrency

- **Enable async**:
  - `@EnableAsync` on a configuration class or main app class.
- **Async methods**:
  - `@Async` on methods of Spring-managed beans.
  - Common return types:
    - `void`
    - `Future<T>`, `CompletableFuture<T>`
    - In reactive contexts: `Mono<T>`, `Flux<T>` (WebFlux/Reactor).
- **Executors (`TaskExecutor`)**:
  - Configure a named `ThreadPoolTaskExecutor` and reference it in `@Async("executorName")`.
  - Important for:
    - Controlling number of threads
    - Queue size
    - Rejection policy, etc.
- **Other async / scheduled mechanisms**:
  - `@Scheduled` for periodic tasks (`fixedRate`, `fixedDelay`, `cron`).
  - Messaging integration (Kafka, RabbitMQ) for background processing.

---

## 10. Application / Bean Lifecycle (Step by Step)

From `main` until the application is ready to serve requests:

- **1. `main` and `SpringApplication.run`**
  - You call:
    ```java
    @SpringBootApplication
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```
  - `SpringApplication` analyzes the main class, determines the app type (servlet / reactive / non-web) and registers default `ApplicationListener`s.

- **2. Environment preparation**
  - Builds the `Environment`:
    - Loads properties from command-line args, environment variables, `application.yml/properties`, external files, etc.
  - Publishes early events such as:
    - `ApplicationStartingEvent`
    - `ApplicationEnvironmentPreparedEvent`
  - Shows the banner if enabled.

- **3. Creation of the `ApplicationContext`**
  - Chooses context implementation:
    - Servlet: `AnnotationConfigServletWebServerApplicationContext`
    - Reactive: `AnnotationConfigReactiveWebServerApplicationContext`
    - Non-web: `AnnotationConfigApplicationContext`
  - Instantiates the context and wires:
    - `Environment`
    - `ApplicationListeners`
    - `ResourceLoader`, etc.

- **4. Registration of configuration classes and auto-configurations**
  - Registers:
    - Your main class (`@SpringBootApplication`) as `@Configuration`.
    - Other configs found via **component scan**.
  - Loads **auto-configurations**:
    - Discovers `@AutoConfiguration` classes via `spring.factories` / `AutoConfiguration.imports`.
    - Evaluates conditions (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.).
    - Only registers auto-configurations whose conditions are satisfied.

- **5. Context `refresh()` – core of the lifecycle**
  - Prepares the `BeanFactory`:
    - Registers default `BeanPostProcessor`s and `BeanFactoryPostProcessor`s.
  - Invokes `BeanFactoryPostProcessor`s:
    - E.g. configuration properties binding, customizations from Boot, etc.
  - Registers custom `BeanPostProcessor`s (provided by you or by auto-config).
  - Initializes message source (i18n) and application event multicaster.
  - **Creates the embedded server** (for web apps):
    - Creates a `ServletWebServerFactory` (Tomcat/Jetty/Undertow) and prepares the `WebServer`.
  - **Instantiates singleton beans**:
    - Calls constructors / dependency injection.
    - Executes `@PostConstruct`, `InitializingBean.afterPropertiesSet`, custom `initMethod`.
    - Applies `BeanPostProcessor` (`postProcessBeforeInitialization` / `postProcessAfterInitialization`).
  - Publishes `ContextRefreshedEvent`.

- **6. Embedded server “online” and startup events**
  - Starts the embedded server and binds the listening port (typically 8080).
  - Publishes:
    - `ApplicationStartedEvent` (context refreshed, before runners).
  - Executes:
    - `ApplicationRunner` and `CommandLineRunner` beans (in order, if `@Order` is used).
  - Publishes:
    - `ApplicationReadyEvent` (context fully initialized, server running, runners executed).

- **7. Steady state and shutdown**
  - From this point:
    - The app serves HTTP requests, processes messages, runs `@Scheduled` tasks, etc.
  - On shutdown:
    - Publishes `ContextClosedEvent`.
    - Calls `@PreDestroy`, `DisposableBean.destroy`, `SmartLifecycle.stop`, etc.

---

## 11. Observability: Actuator, Logs, Metrics

- **Spring Boot Actuator**:
  - Endpoints such as: `/actuator/health`, `/actuator/info`, `/actuator/metrics`, `/actuator/env`, etc.
  - Can integrate with **Micrometer** for Prometheus, Grafana, etc.
- **Logging**:
  - `spring-boot-starter-logging` (Logback by default).
  - Configuration via `application.yml` or `logback-spring.xml`.
- **Tracing / distributed tracing** (modern stack):
  - Spring Observability + Micrometer Tracing, integration with Zipkin/Jaeger/OpenTelemetry.

---

## 12. Testing with Spring Boot

- Unit tests:
  - JUnit + Mockito/MockK, without loading the Spring context (fast).
- Integration tests:
  - `@SpringBootTest` – loads the full application context.
  - `@WebMvcTest` – focuses on the web layer (controllers, filters).
  - `@DataJpaTest` – focuses on JPA repositories (usually with in-memory DB).
- Helper tools:
  - `TestRestTemplate` / `WebTestClient` for endpoint testing.
  - `@Sql` / `@SqlGroup` to prepare data.

---

## 13. Best Practices (Summary)

- **Layered architecture**:
  - Controller → Service → Repository → Domain
  - Avoid business logic in controllers or repositories.
- **Typed configuration**:
  - Prefer `@ConfigurationProperties` over scattered `@Value` usage.
- **Keep beans cohesive**:
  - Small, focused classes with a single responsibility.
- **Centralized exception handling**:
  - `@ControllerAdvice` for REST APIs.
- **Be careful with transactions and lazy loading**:
  - Avoid lazy access outside a transaction (`LazyInitializationException`).
  - Introduce DTOs when appropriate.
- **Async done responsibly**:
  - Configure an appropriate `TaskExecutor`.
  - Avoid blocking threads in reactive or async code.

---

## 14. Ideas for Examples in this Repository

- Simple REST API with:
  - Controllers, Services, Repositories (JPA)
  - Validation, global error handling
  - Basic security (JWT or Basic Auth)
- Asynchronous executions:
  - `@Async` methods with `CompletableFuture`
  - `@Scheduled` jobs
- Observability:
  - Actuator + basic metrics
- Tests:
  - `@WebMvcTest` for controllers
  - `@DataJpaTest` for repositories
  - `@SpringBootTest` for end-to-end flows

---

**Goal**: this `README` serves as a **mental map** of the Spring/Spring Boot ecosystem. As you add examples to this repository, try to reference them in the sections above to build a coherent and concise study material.

## Spring & Spring Boot – Concise Guide to Key Concepts

English version · Versão em português: see `README.pt-BR.md`.

This repository gathers **notes and practical examples** about the main concepts of **Spring Framework** and **Spring Boot**: core modules, commonly used libraries, lifecycle, configuration, async execution, testing, and best practices.

---

## 1. Overview

- **Spring Framework**: Java development platform focused on:
  - **Inversion of Control (IoC)** and **Dependency Injection (DI)**
  - **Aspect-Oriented Programming (AOP)**
  - Abstractions for **data**, **transactions**, **messaging**, **security**, etc.
- **Spring Boot**: a layer on top of Spring that:
  - Simplifies application **bootstrap** (very little manual configuration)
  - Uses **auto‑configuration** and **convention over configuration**
  - Embeds a server (Tomcat/Jetty/Undertow) and packages everything into a **fat jar**

---

## 2. Spring Core (Core / Context / Beans)

- **IoC Container**:
  - Manages the lifecycle of **beans** (objects managed by Spring).
  - Configuration via:
    - Annotations (`@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`…)
    - Java config (`@Bean` methods in `@Configuration` classes)
    - (Legacy) XML.
- **Most common scopes**:
  - `singleton` (default) – one instance per `ApplicationContext`
  - `prototype` – new instance at each injection
  - Web: `request`, `session`, `application`, `websocket`
- **Dependency injection**:
  - `@Autowired`, constructors, setters
  - Best practice: prefer **constructor injection**.

---

## 3. Spring Boot – Basic Structure

- **Main class**:
  - `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
  - `main` method with `SpringApplication.run(...)`.
- **Auto-configuration**:
  - Based on **classpath + properties** (e.g., if `spring-boot-starter-web` is present, it configures MVC + Tomcat).
  - Can be adjusted with `spring.autoconfigure.exclude` or specific annotations.
- **Profiles**:
  - `@Profile("dev")`, `@Profile("prod")`, etc.
  - Files: `application-dev.yml`, `application-prod.yml`.

---

## 4. Main Starters / Libraries

- **Web / APIs**:
  - `spring-boot-starter-web` – MVC, REST, Jackson, Tomcat.
  - `spring-boot-starter-webflux` – reactive programming (Netty).
- **Data**:
  - `spring-boot-starter-data-jpa` – JPA/Hibernate, repositories.
  - `spring-boot-starter-jdbc` – traditional JDBC with `JdbcTemplate`.
  - `spring-boot-starter-data-mongodb`, `data-redis`, etc.
- **Security**:
  - `spring-boot-starter-security` – authentication, authorization, filters, CSRF.
- **Messaging**:
  - `spring-boot-starter-amqp` (RabbitMQ), `spring-kafka`, `spring-cloud-stream`.
- **Observability**:
  - `spring-boot-starter-actuator` – metrics, health checks, info, etc.
- **Template engines / views**:
  - `spring-boot-starter-thymeleaf`, `spring-boot-starter-mustache`.
- **Testing**:
  - `spring-boot-starter-test` – JUnit, Mockito/MockK, AssertJ, Spring Test, etc.

---

## 5. Configuration: Properties / YAML / Profiles

- Main file:
  - `application.properties` or `application.yml`.
- Configuration sources (simplified order of precedence):
  - Environment variables
  - Command-line arguments
  - External files
  - Packaged file (`application.yml` inside the jar)
- **Property binding**:
  - `@Value("${my.config}")`
  - `@ConfigurationProperties(prefix = "app")` + POJO class (typed binding, recommended).

---

## 6. Web: Spring MVC & Controllers

- **REST Controller**:
  - `@RestController` + `@RequestMapping` (or `@GetMapping`, `@PostMapping`…)
- **Validation**:
  - `@Valid` + Bean Validation (`@NotNull`, `@Size`…)
  - Custom messages via `messages.properties`.
- **Error handling**:
  - `@ControllerAdvice` + `@ExceptionHandler`
  - Global exception handling and standardized responses (e.g., `ErrorResponse`).
- **JSON serialization**:
  - `Jackson` by default (Web MVC).
  - Customization via `ObjectMapper` or `Jackson2ObjectMapperBuilder`.

---

## 7. Data Access (JPA, Transactions, etc.)

- **Spring Data JPA**:
  - `Repository` / `CrudRepository` / `JpaRepository` interfaces.
  - Query methods (`findByEmail`, `findByStatusAndType`…).
  - `@Query` for JPQL / native SQL.
- **Transactions**:
  - `@Transactional` on service or repository methods/classes.
  - Configuration of:
    - Propagation (`REQUIRED`, `REQUIRES_NEW`…)
    - Isolation (`READ_COMMITTED`, `REPEATABLE_READ`…)
    - Timeouts and read-only.
- **Migrations**:
  - Integration with `Flyway` or `Liquibase` for schema versioning.

---

## 8. Security with Spring Security (Quick View)

- Filter-based pipelines:
  - Authentication, authorization, CSRF, security headers, etc.
- Common authentication methods:
  - **Form login**, **HTTP Basic**, **JWT**, **OAuth2/OpenID Connect**.
- Modern configuration (Spring Security 5.7+):
  - `SecurityFilterChain` beans instead of extending `WebSecurityConfigurerAdapter`.
- Useful annotations:
  - `@EnableMethodSecurity` / `@PreAuthorize`, `@PostAuthorize`.

---

## 9. Asynchronous Execution and Concurrency

- **Enable async**:
  - `@EnableAsync` on a configuration class or main app class.
- **Async methods**:
  - `@Async` on methods of Spring-managed beans.
  - Common return types:
    - `void`
    - `Future<T>`, `CompletableFuture<T>`
    - In reactive contexts: `Mono<T>`, `Flux<T>` (WebFlux/Reactor).
- **Executors (`TaskExecutor`)**:
  - Configure a named `ThreadPoolTaskExecutor` and reference it in `@Async("executorName")`.
  - Important for:
    - Controlling number of threads
    - Queue size
    - Rejection policy, etc.
- **Other async / scheduled mechanisms**:
  - `@Scheduled` for periodic tasks (`fixedRate`, `fixedDelay`, `cron`).
  - Messaging integration (Kafka, RabbitMQ) for background processing.

---

## 10. Application / Bean Lifecycle (Step by Step)

From `main` until the application is ready to serve requests:

- **1. `main` and `SpringApplication.run`**
  - You call:
    ```java
    @SpringBootApplication
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```
  - `SpringApplication` analyzes the main class, determines the app type (servlet / reactive / non-web) and registers default `ApplicationListener`s.

- **2. Environment preparation**
  - Builds the `Environment`:
    - Loads properties from command-line args, environment variables, `application.yml/properties`, external files, etc.
  - Publishes early events such as:
    - `ApplicationStartingEvent`
    - `ApplicationEnvironmentPreparedEvent`
  - Shows the banner if enabled.

- **3. Creation of the `ApplicationContext`**
  - Chooses context implementation:
    - Servlet: `AnnotationConfigServletWebServerApplicationContext`
    - Reactive: `AnnotationConfigReactiveWebServerApplicationContext`
    - Non-web: `AnnotationConfigApplicationContext`
  - Instantiates the context and wires:
    - `Environment`
    - `ApplicationListeners`
    - `ResourceLoader`, etc.

- **4. Registration of configuration classes and auto-configurations**
  - Registers:
    - Your main class (`@SpringBootApplication`) as `@Configuration`.
    - Other configs found via **component scan**.
  - Loads **auto-configurations**:
    - Discovers `@AutoConfiguration` classes via `spring.factories` / `AutoConfiguration.imports`.
    - Evaluates conditions (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.).
    - Only registers auto-configurations whose conditions are satisfied.

- **5. Context `refresh()` – core of the lifecycle**
  - Prepares the `BeanFactory`:
    - Registers default `BeanPostProcessor`s and `BeanFactoryPostProcessor`s.
  - Invokes `BeanFactoryPostProcessor`s:
    - E.g. configuration properties binding, customizations from Boot, etc.
  - Registers custom `BeanPostProcessor`s (provided by you or by auto-config).
  - Initializes message source (i18n) and application event multicaster.
  - **Creates the embedded server** (for web apps):
    - Creates a `ServletWebServerFactory` (Tomcat/Jetty/Undertow) and prepares the `WebServer`.
  - **Instantiates singleton beans**:
    - Calls constructors / dependency injection.
    - Executes `@PostConstruct`, `InitializingBean.afterPropertiesSet`, custom `initMethod`.
    - Applies `BeanPostProcessor` (`postProcessBeforeInitialization` / `postProcessAfterInitialization`).
  - Publishes `ContextRefreshedEvent`.

- **6. Embedded server “online” and startup events**
  - Starts the embedded server and binds the listening port (typically 8080).
  - Publishes:
    - `ApplicationStartedEvent` (context refreshed, before runners).
  - Executes:
    - `ApplicationRunner` and `CommandLineRunner` beans (in order, if `@Order` is used).
  - Publishes:
    - `ApplicationReadyEvent` (context fully initialized, server running, runners executed).

- **7. Steady state and shutdown**
  - From this point:
    - The app serves HTTP requests, processes messages, runs `@Scheduled` tasks, etc.
  - On shutdown:
    - Publishes `ContextClosedEvent`.
    - Calls `@PreDestroy`, `DisposableBean.destroy`, `SmartLifecycle.stop`, etc.

---

## 11. Observability: Actuator, Logs, Metrics

- **Spring Boot Actuator**:
  - Endpoints such as: `/actuator/health`, `/actuator/info`, `/actuator/metrics`, `/actuator/env`, etc.
  - Can integrate with **Micrometer** for Prometheus, Grafana, etc.
- **Logging**:
  - `spring-boot-starter-logging` (Logback by default).
  - Configuration via `application.yml` or `logback-spring.xml`.
- **Tracing / distributed tracing** (modern stack):
  - Spring Observability + Micrometer Tracing, integration with Zipkin/Jaeger/OpenTelemetry.

---

## 12. Testing with Spring Boot

- Unit tests:
  - JUnit + Mockito/MockK, without loading the Spring context (fast).
- Integration tests:
  - `@SpringBootTest` – loads the full application context.
  - `@WebMvcTest` – focuses on the web layer (controllers, filters).
  - `@DataJpaTest` – focuses on JPA repositories (usually with in-memory DB).
- Helper tools:
  - `TestRestTemplate` / `WebTestClient` for endpoint testing.
  - `@Sql` / `@SqlGroup` to prepare data.

---

## 13. Best Practices (Summary)

- **Layered architecture**:
  - Controller → Service → Repository → Domain
  - Avoid business logic in controllers or repositories.
- **Typed configuration**:
  - Prefer `@ConfigurationProperties` over scattered `@Value` usage.
- **Keep beans cohesive**:
  - Small, focused classes with a single responsibility.
- **Centralized exception handling**:
  - `@ControllerAdvice` for REST APIs.
- **Be careful with transactions and lazy loading**:
  - Avoid lazy access outside a transaction (`LazyInitializationException`).
  - Introduce DTOs when appropriate.
- **Async done responsibly**:
  - Configure an appropriate `TaskExecutor`.
  - Avoid blocking threads in reactive or async code.

---

## 14. Ideas for Examples in this Repository

- Simple REST API with:
  - Controllers, Services, Repositories (JPA)
  - Validation, global error handling
  - Basic security (JWT or Basic Auth)
- Asynchronous executions:
  - `@Async` methods with `CompletableFuture`
  - `@Scheduled` jobs
- Observability:
  - Actuator + basic metrics
- Tests:
  - `@WebMvcTest` for controllers
  - `@DataJpaTest` for repositories
  - `@SpringBootTest` for end-to-end flows

---

**Goal**: this `README` serves as a **mental map** of the Spring/Spring Boot ecosystem. As you add examples to this repository, try to reference them in the sections above to build a coherent and concise study material.

## Spring & Spring Boot – Concise Guide to Key Concepts

This repository gathers **notes and practical examples** about the main concepts of **Spring Framework** and **Spring Boot**: core modules, commonly used libraries, lifecycle, configuration, async execution, testing, and best practices.

---

## 1. Overview

- **Spring Framework**: Java development platform focused on:
  - **Inversion of Control (IoC)** and **Dependency Injection (DI)**
  - **Aspect-Oriented Programming (AOP)**
  - Abstractions for **data**, **transactions**, **messaging**, **security**, etc.
- **Spring Boot**: a layer on top of Spring that:
  - Simplifies application **bootstrap** (very little manual configuration)
  - Uses **auto‑configuration** and **convention over configuration**
  - Embeds a server (Tomcat/Jetty/Undertow) and packages everything into a **fat jar**

---

## 2. Spring Core (Core / Context / Beans)

- **IoC Container**:
  - Manages the lifecycle of **beans** (objects managed by Spring).
  - Configuration via:
    - Annotations (`@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`…)
    - Java config (`@Bean` methods in `@Configuration` classes)
    - (Legacy) XML.
- **Most common scopes**:
  - `singleton` (default) – one instance per `ApplicationContext`
  - `prototype` – new instance at each injection
  - Web: `request`, `session`, `application`, `websocket`
- **Injeção de dependência**:
  - `@Autowired`, constructors, setters
  - Best practice: prefer **constructor injection**.

---

## 3. Spring Boot – Basic Structure

- **Main class**:
  - `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
  - `main` method with `SpringApplication.run(...)`.
- **Auto-configuração**:
  - Baseada em **classpath + propriedades** (ex.: se tem `spring-boot-starter-web`, configura MVC + Tomcat).
  - Pode ser ajustada com `spring.autoconfigure.exclude` ou anotações específicas.
- **Perfis (`profiles`)**:
  - `@Profile("dev")`, `@Profile("prod")`, etc.
  - Arquivos: `application-dev.yml`, `application-prod.yml`.

---

## 4. Principais Starters / Bibliotecas

- **Web / APIs**:
  - `spring-boot-starter-web` – MVC, REST, Jackson, Tomcat.
  - `spring-boot-starter-webflux` – programação reativa (Netty).
- **Dados**:
  - `spring-boot-starter-data-jpa` – JPA/Hibernate, repositórios.
  - `spring-boot-starter-jdbc` – JDBC tradicional com `JdbcTemplate`.
  - `spring-boot-starter-data-mongodb`, `data-redis`, etc.
- **Segurança**:
  - `spring-boot-starter-security` – autenticação, autorização, filtros, CSRF.
- **Mensageria**:
  - `spring-boot-starter-amqp` (RabbitMQ), `spring-kafka`, `spring-cloud-stream`.
- **Observabilidade**:
  - `spring-boot-starter-actuator` – métricas, health checks, info, etc.
- **Template engines / views**:
  - `spring-boot-starter-thymeleaf`, `spring-boot-starter-mustache`.
- **Testes**:
  - `spring-boot-starter-test` – JUnit, Mockito/MockK, AssertJ, Spring Test, etc.

---

## 5. Configuration: Properties / YAML / Profiles

- Main file:
  - `application.properties` or `application.yml`.
- Origens de configuração (ordem de prioridade simplificada):
  - Variáveis de ambiente
  - Linha de comando
  - Arquivos externos
  - Arquivo empacotado (`application.yml` dentro do jar)
- **Binding de propriedades**:
  - `@Value("${minha.config}")`
  - `@ConfigurationProperties(prefix = "app")` + classe POJO (tipada, recomendada).

---

## 6. Web: Spring MVC & Controllers

- **Controller REST**:
  - `@RestController` + `@RequestMapping` (ou `@GetMapping`, `@PostMapping`…)
- **Validação**:
  - `@Valid` + Bean Validation (`@NotNull`, `@Size`…)
  - Mensagens customizáveis via `messages.properties`.
- **Tratamento de erros**:
  - `@ControllerAdvice` + `@ExceptionHandler`
  - Tratamento global de exceções e resposta padronizada (`ErrorResponse`).
- **Serialização JSON**:
  - `Jackson` por padrão (web MVC).
  - Customização via `ObjectMapper` ou `Jackson2ObjectMapperBuilder`.

---

## 7. Acesso a Dados (JPA, Transactions, etc.)

- **Spring Data JPA**:
  - Interfaces `Repository` / `CrudRepository` / `JpaRepository`.
  - Query methods (`findByEmail`, `findByStatusAndType`…).
  - `@Query` para JPQL / SQL nativo.
- **Transações**:
  - `@Transactional` em métodos/classe de serviço ou repositório.
  - Configuração de:
    - Propagation (`REQUIRED`, `REQUIRES_NEW`…)
    - Isolation (`READ_COMMITTED`, `REPEATABLE_READ`…)
    - Timeouts e read-only.
- **Migrations**:
  - Integração com `Flyway` ou `Liquibase` para versionamento de schema.

---

## 8. Segurança com Spring Security (Visão Rápida)

- Pipelines baseadas em **filters**:
  - Autenticação, autorização, CSRF, headers de segurança…
- Formas comuns de autenticação:
  - **Form login**, **HTTP Basic**, **JWT**, **OAuth2/OpenID Connect**.
- Configuração moderna (Spring Security 5.7+):
  - Beans `SecurityFilterChain` ao invés de herdar `WebSecurityConfigurerAdapter`.
- Anotações úteis:
  - `@EnableMethodSecurity` / `@PreAuthorize`, `@PostAuthorize`.

---

## 9. Execuções Assíncronas e Concorrência

- **Habilitar async**:
  - `@EnableAsync` em classe de configuração ou app principal.
- **Métodos assíncronos**:
  - `@Async` em métodos de beans gerenciados pelo Spring.
  - Retornos comuns:
    - `void`
    - `Future<T>`, `CompletableFuture<T>`
    - Em contextos reativos: `Mono<T>`, `Flux<T>` (WebFlux/Reactor).
- **Executores (`TaskExecutor`)**:
  - Configurar `ThreadPoolTaskExecutor` nomeado e referenciar em `@Async("nomeDoExecutor")`.
  - Importante para:
    - Controlar nº de threads
    - Tamanho de fila
    - Política de rejeição, etc.
- **Outros mecanismos assíncronos / agendados**:
  - `@Scheduled` para tarefas periódicas (`fixedRate`, `fixedDelay`, `cron`).
  - Integração com mensageria (Kafka, RabbitMQ) para processamento em background.

---

## 10. Application / Bean Lifecycle (Step by Step)

From `main` until the application is ready to serve requests:

- **1. `main` and `SpringApplication.run`**
  - You call:
    ```java
    @SpringBootApplication
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```
  - `SpringApplication` analyzes the main class, determines the app type (servlet / reactive / non-web) and registers default `ApplicationListener`s.

- **2. Environment preparation**
  - Builds the `Environment`:
    - Loads properties from command-line args, environment variables, `application.yml/properties`, external files, etc.
  - Publishes early events such as:
    - `ApplicationStartingEvent`
    - `ApplicationEnvironmentPreparedEvent`
  - Shows the banner if enabled.

- **3. Creation of the `ApplicationContext`**
  - Chooses context implementation:
    - Servlet: `AnnotationConfigServletWebServerApplicationContext`
    - Reactive: `AnnotationConfigReactiveWebServerApplicationContext`
    - Non-web: `AnnotationConfigApplicationContext`
  - Instantiates the context and wires:
    - `Environment`
    - `ApplicationListeners`
    - `ResourceLoader`, etc.

- **4. Registration of configuration classes and auto-configurations**
  - Registers:
    - Your main class (`@SpringBootApplication`) as `@Configuration`.
    - Other configs found via **component scan**.
  - Loads **auto-configurations**:
    - Discovers `@AutoConfiguration` classes via `spring.factories` / `AutoConfiguration.imports`.
    - Evaluates conditions (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.).
    - Only registers auto-configurations whose conditions are satisfied.

- **5. Context `refresh()` – core of the lifecycle**
  - Prepares the `BeanFactory`:
    - Registers default `BeanPostProcessor`s and `BeanFactoryPostProcessor`s.
  - Invokes `BeanFactoryPostProcessor`s:
    - E.g. configuration properties binding, customizations from Boot, etc.
  - Registers custom `BeanPostProcessor`s (provided by you or by auto-config).
  - Initializes message source (i18n) and application event multicaster.
  - **Creates the embedded server** (for web apps):
    - Creates a `ServletWebServerFactory` (Tomcat/Jetty/Undertow) and prepares the `WebServer`.
  - **Instantiates singleton beans**:
    - Calls constructors / dependency injection.
    - Executes `@PostConstruct`, `InitializingBean.afterPropertiesSet`, custom `initMethod`.
    - Applies `BeanPostProcessor` (`postProcessBeforeInitialization` / `postProcessAfterInitialization`).
  - Publishes `ContextRefreshedEvent`.

- **6. Embedded server “online” and startup events**
  - Starts the embedded server and binds the listening port (typically 8080).
  - Publishes:
    - `ApplicationStartedEvent` (context refreshed, before runners).
  - Executes:
    - `ApplicationRunner` and `CommandLineRunner` beans (in order, if `@Order` is used).
  - Publishes:
    - `ApplicationReadyEvent` (context fully initialized, server running, runners executed).

- **7. Steady state and shutdown**
  - From this point:
    - The app serves HTTP requests, processes messages, runs `@Scheduled` tasks, etc.
  - On shutdown:
    - Publishes `ContextClosedEvent`.
    - Calls `@PreDestroy`, `DisposableBean.destroy`, `SmartLifecycle.stop`, etc.

---

## 11. Observabilidade: Actuator, Logs, Métricas

- **Spring Boot Actuator**:
  - Endpoints como: `/actuator/health`, `/actuator/info`, `/actuator/metrics`, `/actuator/env`, etc.
  - Pode integrar com **Micrometer** para Prometheus, Grafana, etc.
- **Logging**:
  - `spring-boot-starter-logging` (Logback por padrão).
  - Configuração via `application.yml` ou `logback-spring.xml`.
- **Tracing / distributed tracing** (stack moderna):
  - Spring Observability + Micrometer Tracing, integração com Zipkin/Jaeger/Otel.

---

## 12. Testes com Spring Boot

- Testes de unidade:
  - JUnit + Mockito/MockK, sem subir contexto (veloz).
- Testes de integração:
  - `@SpringBootTest` – sobe o contexto completo.
  - `@WebMvcTest` – foca em camada web (controllers, filters).
  - `@DataJpaTest` – foca em repositórios JPA (usa banco em memória por padrão).
- Ferramentas auxiliares:
  - `TestRestTemplate` / `WebTestClient` para testar endpoints.
  - `@Sql` / `@SqlGroup` para preparar dados.

---

## 13. Boas Práticas (Resumo)

- **Estrutura de camadas**:
  - Controller → Service → Repository → Domain
  - Evitar lógica de negócio em controllers ou repositórios.
- **Configuração tipada**:
  - Preferir `@ConfigurationProperties` a `@Value` espalhado.
- **Manter beans coesos**:
  - Classes pequenas, focadas, responsabilidade única.
- **Tratar exceções de forma centralizada**:
  - `@ControllerAdvice` para APIs REST.
- **Cuidado com transações e lazy loading**:
  - Evitar acesso lazy fora de transação (`LazyInitializationException`).
  - Introduzir DTOs quando necessário.
- **Async com responsabilidade**:
  - Configurar `TaskExecutor` apropriado.
  - Evitar bloquear threads em código reativo ou async.

---

## 14. Ideias para Exemplos neste Repositório

- API REST simples com:
  - Controllers, Services, Repositories (JPA)
  - Validação, tratamento global de erros
  - Segurança básica (JWT ou Basic Auth)
- Execuções assíncronas:
  - Métodos `@Async` com `CompletableFuture`
  - Jobs `@Scheduled`
- Observabilidade:
  - Actuator + métricas básicas
- Testes:
  - `@WebMvcTest` para controllers
  - `@DataJpaTest` para repositórios
  - `@SpringBootTest` para fluxo de ponta a ponta

---

**Objetivo**: este `README` serve como um **mapa mental** do ecossistema Spring/Spring Boot. À medida que você for adicionando exemplos neste repositório, tente referenciá-los nas seções acima para criar um material de estudo coerente e enxuto.
