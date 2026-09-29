# Spring 02: Spring Boot Essentials

Spring is the framework. Spring Boot is the layer on top that configures Spring for you. An intern often cannot tell them apart. Ask them.

## 1. What Boot adds

| Feature | Meaning |
|---------|---------|
| Starters | One dependency pulls a tested set of libraries, for example `spring-boot-starter-web` |
| Auto-configuration | Beans are created automatically, based on the classpath and properties |
| Embedded server | Tomcat runs inside the JAR. You run `java -jar app.jar` |
| Externalized configuration | Properties from files, environment variables, and command-line arguments |
| Actuator | Health, metrics, and info endpoints for production |

## 2. `@SpringBootApplication`

```java
@SpringBootApplication
public class ShopApplication {
    public static void main(String[] args) {
        SpringApplication.run(ShopApplication.class, args);
    }
}
```

It combines three annotations:

- `@SpringBootConfiguration`: this class is a configuration class.
- `@EnableAutoConfiguration`: turn on auto-configuration.
- `@ComponentScan`: scan this package and all sub-packages.

Trap: component scan starts at the package of this class. A `@Service` in a sibling package (outside) is not found.

## 3. How auto-configuration works

Boot 3 lists auto-configuration classes in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 2 used `spring.factories`). Each class uses conditions:

```java
@AutoConfiguration
@ConditionalOnClass(DataSource.class)          // only if the class is on the classpath
@ConditionalOnMissingBean(DataSource.class)    // only if you did not define your own
public class DataSourceAutoConfiguration { /* ... */ }
```

Key idea: your own bean always wins, because of `@ConditionalOnMissingBean`. To see which auto-configurations ran, start the app with `--debug` and read the condition evaluation report.

## 4. Configuration properties

`application.yml`:

```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/shop
    username: ${DB_USER}
    password: ${DB_PASSWORD}
app:
  payment:
    base-url: https://pay.example.com
    timeout: 5s
    retries: 3
```

Type-safe binding (preferred over many `@Value` fields):

```java
@ConfigurationProperties(prefix = "app.payment")
@Validated
public record PaymentProperties(
        @NotBlank String baseUrl,
        Duration timeout,
        @Min(0) int retries) {}

@SpringBootApplication
@ConfigurationPropertiesScan
public class ShopApplication { /* ... */ }
```

`@Value("${app.payment.base-url}")` works for a single value. It has no type grouping and no validation.

Relaxed binding: `base-url`, `baseUrl`, and the environment variable `APP_PAYMENT_BASEURL` all map to the same property.

## 5. Property order

Boot reads many sources. A later source in this short list overrides an earlier one:

1. `application.yml` inside the JAR
2. `application-{profile}.yml` inside the JAR
3. `application.yml` outside the JAR (next to it)
4. Environment variables
5. Command-line arguments (`--server.port=9090`)

Rule for secrets: never put them in `application.yml` in Git. Use environment variables or a secret manager.

## 6. Profiles

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:shop
```

Activate with `SPRING_PROFILES_ACTIVE=dev` or `--spring.profiles.active=dev`. Beans can also depend on a profile:

```java
@Bean @Profile("dev")
public PaymentClient fakePaymentClient() { return new FakePaymentClient(); }
```

## 7. Layered application structure

```
com.example.shop
 ├── ShopApplication.java
 ├── order
 │    ├── OrderController.java     web: HTTP in, DTO out
 │    ├── OrderService.java        business rules, transactions
 │    ├── OrderRepository.java     data access
 │    ├── Order.java               JPA entity
 │    └── dto/                     request and response records
 └── common
      └── error/GlobalExceptionHandler.java
```

Package by feature (`order`, `user`) scales better than package by layer (`controllers`, `services`). Both are common. Follow the team's existing structure.

Rules:

- The controller does not contain business logic and does not call the repository directly.
- The service does not know about HTTP (`HttpServletRequest`, `ResponseEntity`).
- The entity does not leave the service layer. The controller returns DTOs (Spring lesson 03).

## 8. Actuator

Add `spring-boot-starter-actuator`. Useful endpoints: `/actuator/health`, `/actuator/info`, `/actuator/metrics`. Only `health` is exposed over HTTP by default. Do not expose `env` or `heapdump` publicly, because they leak secrets.

## 9. Logging

Boot uses SLF4J with Logback.

```java
private static final Logger log = LoggerFactory.getLogger(OrderService.class);
log.info("Order {} placed by user {}", orderId, userId);   // placeholders, not string concatenation
log.error("Payment failed for order {}", orderId, ex);     // exception as the last argument
```

Lombok `@Slf4j` generates the `log` field. Never log passwords, tokens, or full card numbers.

## Try it

1. Create `PaymentProperties` as a record with validation. Start the app with an empty `base-url` and read the error.
2. Override `app.payment.retries` with an environment variable and with a command-line argument. Note which one wins.
3. Start with `--debug` and find `DataSourceAutoConfiguration` in the report.
4. Define your own `ObjectMapper` bean and explain what happens to Boot's default one.
