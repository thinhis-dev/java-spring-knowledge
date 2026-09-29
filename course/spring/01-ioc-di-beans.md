# Spring 01: IoC, DI, and Beans

You use Spring every day in Kotlin. This lesson gives you the vocabulary and the internal model, so that you can explain it to an intern and ask good follow-up questions.

## 1. IoC and DI

Inversion of control (IoC) means that a framework creates and connects your objects, instead of your code. Dependency injection (DI) is the way Spring does it: it passes dependencies into an object from outside.

The Spring container is the `ApplicationContext`. At startup, it:

1. Scans packages for classes with stereotype annotations and reads `@Bean` methods.
2. Builds a bean definition for each one.
3. Creates the beans in dependency order and injects the dependencies.
4. Wraps some beans in proxies (for `@Transactional`, `@Async`, and similar).
5. Runs lifecycle callbacks.

A bean is an object that the container creates and manages.

## 2. Declaring beans

Stereotype annotations on your own classes:

| Annotation | Layer | Extra behavior |
|-----------|-------|----------------|
| `@Component` | Generic | None |
| `@Service` | Business logic | None, only intent |
| `@Repository` | Data access | Translates persistence exceptions into `DataAccessException` |
| `@Controller` / `@RestController` | Web | Handles HTTP requests |
| `@Configuration` | Configuration | Holds `@Bean` methods |

`@Bean` methods for classes that you do not own:

```java
@Configuration
public class HttpConfig {
    @Bean
    public RestClient githubClient(RestClient.Builder builder) {
        return builder.baseUrl("https://api.github.com").build();
    }
}
```

`@Configuration` classes are proxied. If one `@Bean` method calls another `@Bean` method, Spring returns the same singleton, not a new object.

## 3. Injection types

```java
@Service
public class OrderService {
    private final OrderRepository repo;
    private final PaymentClient payments;

    // Constructor injection. With one constructor, @Autowired is not needed.
    public OrderService(OrderRepository repo, PaymentClient payments) {
        this.repo = repo;
        this.payments = payments;
    }
}
```

With Lombok, `@RequiredArgsConstructor` generates this constructor for all `final` fields.

| Type | Use it? | Reason |
|------|---------|--------|
| Constructor | Yes, always the default | Fields can be `final`. Dependencies are explicit. Easy to test with `new`. A missing dependency stops the startup at once. |
| Setter | Rarely | For optional dependencies |
| Field (`@Autowired private X x;`) | No | Hides dependencies, no `final`, needs reflection or Spring in tests |

An intern who uses field injection everywhere is a common case. Ask why constructor injection is better.

## 4. Choosing between beans of the same type

If two beans implement `PaymentClient`, injection fails with `NoUniqueBeanDefinitionException`. Fixes:

```java
@Primary @Component class StripeClient implements PaymentClient {}  // the default choice

public OrderService(@Qualifier("paypalClient") PaymentClient client) {} // choose by name

public OrderService(List<PaymentClient> clients) {}      // inject all of them
public OrderService(Map<String, PaymentClient> clients) {} // bean name to bean
```

## 5. Bean scopes

| Scope | Instances | Use |
|-------|-----------|-----|
| `singleton` (default) | One per container | Stateless services, almost everything |
| `prototype` | New one each time it is requested | Stateful helper objects |
| `request` | One per HTTP request | Request data (web only) |
| `session` | One per HTTP session | Session data (web only) |

A singleton bean is shared by all request threads. It must not hold per-request mutable state (Java lesson 07).

Trap: if you inject a `prototype` bean into a singleton, the singleton gets one prototype instance at creation and keeps it. Use `ObjectProvider<T>` to get a new instance each time.

## 6. Lifecycle

```java
@Component
public class CacheWarmer {
    @PostConstruct   // after dependencies are injected
    void warm() { /* ... */ }

    @PreDestroy      // before the container closes
    void flush() { /* ... */ }
}
```

In Boot 3, these annotations come from `jakarta.annotation`, not `javax.annotation`.

For work after the whole application starts, implement `ApplicationRunner` or listen to `ApplicationReadyEvent`.

## 7. Proxies and the self-invocation trap

For `@Transactional`, `@Async`, `@Cacheable`, and `@PreAuthorize`, Spring does not inject your object. It injects a proxy that wraps your object. The proxy runs extra code before and after the call. Spring uses a JDK dynamic proxy for interfaces and a CGLIB subclass proxy for classes.

```
Controller ──> [Proxy: begin transaction] ──> OrderService.placeOrder() ──> [Proxy: commit]
```

Consequence: a call from inside the same object does not go through the proxy.

```java
@Service
public class OrderService {
    public void placeAll(List<Order> orders) {
        orders.forEach(this::place); // this.place(): no proxy, no transaction
    }

    @Transactional
    public void place(Order o) { /* ... */ }
}
```

Also, the annotation has no effect on `private` methods, and on `final` methods and classes with CGLIB. Kotlin classes are `final` by default. That is the reason for the `kotlin-spring` (all-open) compiler plugin in your Kotlin project. Java classes are open, so Java does not need it.

Fixes: move the method to another bean, or put `@Transactional` on the outer method.

## 8. Circular dependencies

A needs B and B needs A. With constructor injection, startup fails. Spring Boot 2.6 and later also forbids circular references by default. The correct fix is a redesign: extract the shared logic into a third bean. `@Lazy` hides the problem.

## 9. AOP in one paragraph

Aspect-oriented programming (AOP) adds cross-cutting behavior (logging, transactions, security, metrics) without editing each method. An aspect has advice (the code to run) and a pointcut (where to run it). Spring AOP implements this with the proxies from section 7.

```java
@Aspect @Component
public class TimingAspect {
    @Around("@annotation(com.example.Timed)")
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try { return pjp.proceed(); }
        finally { log.info("{} took {} ms", pjp.getSignature(), (System.nanoTime() - start) / 1_000_000); }
    }
}
```

## Try it

1. Create a Boot project (see the README). Add two `PaymentClient` beans and read the startup error. Fix it with `@Primary`, then with `List<PaymentClient>`.
2. Reproduce the self-invocation trap: add a log line inside a `@Transactional` method and print `TransactionSynchronizationManager.isActualTransactionActive()` for both call paths.
3. Print `orderService.getClass()` in a test. Explain the `$$SpringCGLIB$$` part of the name.
