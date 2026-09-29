# Java 05: Lambdas, Streams, and Optional

Kotlin collection functions (`map`, `filter`, `groupBy`) are eager and work directly on lists. Java uses the Stream API, which is lazy and needs a terminal operation.

## 1. Functional interfaces and lambdas

A functional interface has exactly one abstract method. A lambda is an instance of such an interface.

| Interface | Method | Kotlin type |
|-----------|--------|-------------|
| `Supplier<T>` | `T get()` | `() -> T` |
| `Consumer<T>` | `void accept(T)` | `(T) -> Unit` |
| `Function<T, R>` | `R apply(T)` | `(T) -> R` |
| `Predicate<T>` | `boolean test(T)` | `(T) -> Boolean` |
| `BiFunction<T, U, R>` | `R apply(T, U)` | `(T, U) -> R` |
| `UnaryOperator<T>` | `T apply(T)` | `(T) -> T` |
| `Runnable` | `void run()` | `() -> Unit` |

```java
Predicate<String> isBlank = s -> s.isBlank();
Predicate<String> isBlank2 = String::isBlank; // method reference
Function<String, Integer> length = String::length;
Supplier<List<String>> newList = ArrayList::new;
```

If a lambda uses a local variable, the variable must be effectively final (not changed after assignment).

```java
int count = 0;
list.forEach(x -> count++); // compile error
```

## 2. Stream pipeline

```java
List<String> emails = users.stream()        // source
    .filter(User::isActive)                 // intermediate, lazy
    .map(User::getEmail)                    // intermediate, lazy
    .distinct()
    .sorted()
    .toList();                              // terminal: runs the pipeline
```

Rules:

- Nothing runs until a terminal operation (`toList`, `collect`, `forEach`, `count`, `reduce`, `findFirst`, `anyMatch`).
- A stream can be used only once.
- Streams process one element through the whole pipeline before the next element. Short-circuit operations like `findFirst` and `limit` stop early.
- Do not change external state inside `map` or `filter`.

```java
Stream.of("a", "b", "c")
    .peek(s -> System.out.println("saw " + s))
    .filter(s -> !s.equals("a"))
    .findFirst();
// prints: saw a, saw b. It never reaches "c".
```

## 3. Common operations

```java
// flatMap: one-to-many
List<String> allTags = posts.stream().flatMap(p -> p.tags().stream()).toList();

// reduce
int total = orders.stream().map(Order::amount).reduce(0, Integer::sum);

// primitive streams avoid boxing
int total2 = orders.stream().mapToInt(Order::amount).sum();
IntStream.range(0, 5).forEach(System.out::println);

// anyMatch, allMatch, noneMatch
boolean hasAdmin = users.stream().anyMatch(u -> u.role() == Role.ADMIN);
```

## 4. Collectors

```java
import static java.util.stream.Collectors.*;

Map<Role, List<User>> byRole = users.stream().collect(groupingBy(User::role));

Map<Role, Long> countByRole = users.stream().collect(groupingBy(User::role, counting()));

Map<Long, User> byId = users.stream().collect(toMap(User::id, u -> u));
// toMap throws IllegalStateException on a duplicate key. Pass a merge function:
Map<String, User> byEmail = users.stream()
    .collect(toMap(User::email, u -> u, (first, second) -> first));

Map<Boolean, List<User>> split = users.stream().collect(partitioningBy(User::isActive));

String csv = names.stream().collect(joining(", ", "[", "]"));
```

`collect(Collectors.toList())` returns a mutable list in practice. `Stream.toList()` (Java 16+) returns an unmodifiable list.

## 5. When not to use streams

- Simple loops with early `return` or checked exceptions are often clearer as a `for` loop.
- `parallelStream()` is rarely a good idea in a web server. It uses the shared common fork-join pool and can make things slower. Use it only after you measure.

## 6. Optional

`Optional<T>` is a container that holds a value or nothing. It makes "no result" explicit in a return type.

```java
Optional<User> user = userRepository.findById(id); // Spring Data returns Optional

String name = user.map(User::getName).orElse("unknown");
User u = user.orElseThrow(() -> new UserNotFoundException(id));
user.ifPresent(x -> log.info("found {}", x));
```

Rules:

- Use `Optional` as a return type only. Do not use it for fields, method parameters, or collections.
- Never return `null` from a method that returns `Optional`.
- Do not call `get()` without a check. Prefer `orElseThrow()`.
- `orElse(x)` always evaluates `x`. `orElseGet(() -> x)` evaluates `x` only for an empty `Optional`. If `x` is expensive, use `orElseGet`.
- Return an empty collection, not `Optional<List<T>>`.

## Try it

Given `record Order(long id, String customer, String status, int amount)`:

1. Return the total amount of `PAID` orders.
2. Return `Map<String, Integer>`: total amount per customer.
3. Return the customer with the highest total, as `Optional<String>`.
4. Return a `Map<String, List<Long>>`: order IDs grouped by status.
5. Two orders have the same customer. Explain the result of `toMap(Order::customer, Order::amount)` on them.
