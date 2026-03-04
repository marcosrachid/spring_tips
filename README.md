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

## 2. Núcleo do Spring (Core / Context / Beans)

- **Container de IoC**:
  - Gerencia o ciclo de vida de **beans** (objetos gerenciados pelo Spring).
  - Configuração via:
    - Anotações (`@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`…)
    - Java config (`@Bean` em classes `@Configuration`)
    - (Legado) XML.
- **Scopes mais comuns**:
  - `singleton` (default) – uma instância por `ApplicationContext`
  - `prototype` – nova instância a cada injeção
  - Web: `request`, `session`, `application`, `websocket`
- **Injeção de dependência**:
  - `@Autowired`, construtores, setters
  - Boas práticas: preferir **injeção via construtor**.

---

## 3. Spring Boot – Estrutura Básica

- **Classe principal**:
  - `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
  - Método `main` com `SpringApplication.run(...)`.
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

## 5. Configuração: Properties / YAML / Profiles

- Arquivo principal:
  - `application.properties` ou `application.yml`.
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
