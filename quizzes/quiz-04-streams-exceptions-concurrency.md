# Quiz 04: Streams, Exceptions, Concurrency, and the JVM

Level: 4 (medium to hard). Covers: Java lessons 05, 06, and 07. Time: about 75 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

Use this record for Q1 to Q4:

```java
record Order(long id, String customer, String status, int amount) {}
List<Order> orders = ...;
```

## Q1

Write a stream that returns the total amount of orders with status `PAID`.

Answer:

## Q2

Write a stream that returns `Map<String, Integer>`: the total amount per customer.

Answer:

## Q3

Write a stream that returns the name of the customer with the highest total amount, as `Optional<String>`.

Answer:

## Q4

This line throws an exception at runtime. Which one, and why? Fix it.

```java
Map<String, Order> byCustomer = orders.stream()
    .collect(Collectors.toMap(Order::customer, o -> o));
```

Answer:

## Q5

What does this code print? Explain the order.

```java
Stream.of("a", "bb", "ccc", "dddd")
    .filter(s -> { System.out.println("filter " + s); return s.length() > 1; })
    .map(s -> { System.out.println("map " + s); return s.toUpperCase(); })
    .findFirst();
```

Answer:

## Q6

Name four rules for good use of `Optional`. What is the difference between `orElse` and `orElseGet`?

Answer:

## Q7

Compare checked and unchecked exceptions. Why does Spring prefer unchecked exceptions?

Answer:

## Q8

What does this method return, and what happens to the exception?

```java
static int test() {
    try {
        throw new RuntimeException("boom");
    } finally {
        return 2;
    }
}
```

Answer:

## Q9

What is try-with-resources? In which order do the resources close? What is a suppressed exception?

Answer:

## Q10

This does not compile. Why? Fix it.

```java
List<String> contents = paths.stream().map(Files::readString).toList();
```

Answer:

## Q11

This Spring service runs in a web server. What is the bug? Give two fixes, and say which one is best.

```java
@Service
public class VisitCounter {
    private int visits = 0;
    public int visit() { return ++visits; }
}
```

Answer:

## Q12

Is `volatile` enough to fix Q11? Explain the difference between visibility and atomicity.

Answer:

## Q13

Load a user and the user's orders at the same time with `CompletableFuture`, then combine them into a `UserView`. On an error, return `UserView.empty()`.

Answer:

## Q14

What is a deadlock? Name two ways to prevent it.

Answer:

## Q15

Where do these live: a local `int`, a local reference variable, and the object that it points to? Which area is shared between threads?

Answer:

## Q16

Java has a garbage collector. How can a Java application still have a memory leak? Give two examples.

Answer:

## Q17

Why must code call `ThreadLocal.remove()` in a thread pool?

Answer:
