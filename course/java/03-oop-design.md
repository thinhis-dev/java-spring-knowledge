# Java 03: OOP Design

Knowing the syntax is not enough. This lesson covers how to design classes that are easy to change. Spring code depends on these ideas, especially dependency inversion.

## 1. Composition over inheritance

Inheritance couples the child to the internal behavior of the parent. Composition means that a class holds another object and delegates to it.

Classic inheritance bug:

```java
public class CountingSet<E> extends HashSet<E> {
    private int addCount = 0;
    @Override public boolean add(E e) { addCount++; return super.add(e); }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c); // HashSet.addAll calls add(), so each item counts twice
    }
}
```

Composition fix:

```java
public class CountingSet<E> {
    private final Set<E> delegate = new HashSet<>();
    private int addCount = 0;

    public boolean add(E e) { addCount++; return delegate.add(e); }
    public boolean addAll(Collection<? extends E> c) {
        boolean changed = false;
        for (E e : c) changed |= add(e);
        return changed;
    }
    public int addCount() { return addCount; }
}
```

Use inheritance only for a true "is-a" relation with a parent that is designed for extension.

## 2. SOLID

| Letter | Principle | One-line meaning |
|--------|-----------|------------------|
| S | Single responsibility | A class has one reason to change |
| O | Open/closed | Add behavior by adding code, not by editing old code |
| L | Liskov substitution | A subclass must work everywhere its parent works |
| I | Interface segregation | Many small interfaces are better than one big interface |
| D | Dependency inversion | Depend on abstractions, not on concrete classes |

### S: single responsibility

Bad: `UserService` validates input, saves to the database, sends email, and builds a PDF. Good: `UserService` coordinates. `UserRepository`, `EmailSender`, and `PdfRenderer` each do one job.

### O: open/closed with the strategy pattern

Bad:

```java
long discount(Order o) {
    if (o.type().equals("VIP")) return o.total() / 10;
    else if (o.type().equals("STUDENT")) return o.total() / 20;
    return 0; // each new type edits this method
}
```

Good:

```java
interface DiscountPolicy {
    boolean supports(Customer c);
    long discount(Order o);
}

class VipDiscount implements DiscountPolicy { /* ... */ }
class StudentDiscount implements DiscountPolicy { /* ... */ }

class DiscountService {
    private final List<DiscountPolicy> policies;
    DiscountService(List<DiscountPolicy> policies) { this.policies = policies; }

    long discount(Customer c, Order o) {
        return policies.stream()
            .filter(p -> p.supports(c))
            .mapToLong(p -> p.discount(o))
            .max().orElse(0);
    }
}
```

A new type is a new class. If each policy is a Spring `@Component`, Spring injects all of them into `List<DiscountPolicy>`.

### L: Liskov substitution

Classic violation: `Square extends Rectangle`. A `Rectangle` lets you set width and height separately. A `Square` must keep them equal, so code that sets width 2 and height 3 and expects area 6 breaks with a `Square`. Another violation: a subclass that throws `UnsupportedOperationException` for a parent method.

### I: interface segregation

Bad: `interface Worker { void work(); void eat(); }`, then a `Robot` must implement `eat()`. Good: split into `Workable` and `Eater`.

### D: dependency inversion

```java
// Bad: OrderService creates its own dependency
class OrderService {
    private final SmtpEmailSender sender = new SmtpEmailSender();
}

// Good: OrderService depends on an interface, and someone else gives it the implementation
class OrderService {
    private final EmailSender sender;
    OrderService(EmailSender sender) { this.sender = sender; }
}
```

The good version is easy to test (pass a fake `EmailSender`) and easy to change. Dependency injection (DI) is the technique of passing dependencies in from outside. Spring is a DI container: it builds the objects and passes the dependencies (Spring lesson 01).

## 3. Patterns that interviews ask about

### Builder

Java has no named or default arguments, so a builder helps with many optional fields.

```java
public final class HttpRequest {
    private final String url;
    private final String method;
    private final int timeoutMs;

    private HttpRequest(Builder b) {
        this.url = b.url;
        this.method = b.method;
        this.timeoutMs = b.timeoutMs;
    }

    public static Builder builder(String url) { return new Builder(url); }

    public static final class Builder {
        private final String url;
        private String method = "GET";
        private int timeoutMs = 5000;

        private Builder(String url) { this.url = url; }
        public Builder method(String m) { this.method = m; return this; }
        public Builder timeoutMs(int t) { this.timeoutMs = t; return this; }
        public HttpRequest build() { return new HttpRequest(this); }
    }
}

HttpRequest r = HttpRequest.builder("https://api").method("POST").build();
```

Lombok `@Builder` generates this code.

### Factory method

A static method that creates an object and hides the concrete class: `List.of(...)`, `Optional.of(...)`, `LocalDate.of(...)`.

### Singleton

One instance for the whole application. In plain Java, the simplest safe version is an enum:

```java
public enum Config {
    INSTANCE;
    public String get(String key) { /* ... */ return null; }
}
```

In Spring, you almost never write a singleton by hand. Spring beans are singletons by default (one instance per container).

### Strategy

See the `DiscountPolicy` example above. Choose behavior at runtime through an interface.

### Template method

An abstract class defines the steps of an algorithm and lets subclasses fill some steps. Spring's `JdbcTemplate` uses a callback variant of this idea.

### Proxy and decorator

A proxy wraps an object and adds behavior before or after the call. Spring uses proxies for `@Transactional`, `@Cacheable`, and `@Async`. This is important for Spring lesson 01 and 04.

## 4. Code smells to spot in intern code

- God class: one class with 1,000 lines and ten responsibilities.
- Anemic model with logic in the controller.
- `instanceof` chains or `switch` on a type string where polymorphism fits.
- Public mutable fields.
- Static state that changes (global variables).
- `new` of a service inside another service (no DI).
- Boolean flags that change method behavior: `save(user, true, false)`.

## Try it

1. Implement the `DiscountPolicy` strategy with two policies and a test in `main`.
2. Write the `CountingSet` inheritance version, show the double count, then fix it with composition.
3. Write a builder for a `Pizza` with a required size and optional toppings.
