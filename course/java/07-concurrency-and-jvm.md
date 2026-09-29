# Java 07: Concurrency and the JVM

A Spring MVC server handles each request on its own thread. A singleton bean is shared by all these threads. You must know enough concurrency to spot a thread-safety bug in intern code. Kotlin coroutines do not exist in plain Java.

## 1. Threads and executors

```java
Thread t = new Thread(() -> System.out.println("hi"));
t.start(); // start, not run. run() executes on the current thread.
t.join();  // wait for it to finish
```

In real code, use a thread pool:

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f = pool.submit(() -> compute());
Integer result = f.get(); // blocks
pool.shutdown();
```

## 2. CompletableFuture

`CompletableFuture` chains async steps, like a JavaScript `Promise`.

```java
CompletableFuture<User> userF = CompletableFuture.supplyAsync(() -> loadUser(id), pool);
CompletableFuture<List<Order>> ordersF = CompletableFuture.supplyAsync(() -> loadOrders(id), pool);

UserView view = userF.thenCombine(ordersF, UserView::new)
    .exceptionally(ex -> UserView.empty())
    .join();
```

| JavaScript | Java |
|-----------|------|
| `.then(x => y)` | `thenApply` |
| `.then(x => promise)` | `thenCompose` |
| `Promise.all` | `CompletableFuture.allOf` |
| `.catch` | `exceptionally` or `handle` |

## 3. Race conditions

```java
@Service
public class CounterService {
    private int count = 0;               // shared by all request threads
    public void increment() { count++; } // read, add, write: not atomic
}
```

Two threads can read 5 at the same time and both write 6. Fixes, from best to worst for this case:

```java
private final AtomicInteger count = new AtomicInteger(); // lock-free
count.incrementAndGet();

public synchronized void increment() { count++; }       // one thread at a time

private final ReentrantLock lock = new ReentrantLock();  // explicit lock, more features
```

Best fix in a web app: keep no mutable state in singleton beans. Keep state in the database or in local variables.

## 4. Visibility and `volatile`

Each thread can cache a variable. Without synchronization, a thread can miss a write from another thread forever.

```java
private volatile boolean running = true; // writes are visible to all threads at once
```

`volatile` gives visibility, not atomicity. `volatile int count; count++` is still a race.

## 5. Thread-safe collections

- `ConcurrentHashMap`: use `compute`, `merge`, or `computeIfAbsent` for atomic updates.
  ```java
  counts.merge(word, 1, Integer::sum);
  ```
- `CopyOnWriteArrayList`: many reads, rare writes.
- `Collections.synchronizedList`: coarse lock. Iteration still needs manual synchronization.
- `HashMap` shared between threads without a lock can corrupt its internal state.

## 6. Deadlock

Thread A locks X and waits for Y. Thread B locks Y and waits for X. Both wait forever. Prevention: always take locks in the same global order, use timeouts (`tryLock`), or keep critical sections small.

## 7. ThreadLocal

`ThreadLocal<T>` gives each thread its own copy of a value. Spring uses it to store the current transaction and the `SecurityContext`. In a thread pool, always call `remove()` at the end, because the thread is reused and the old value leaks into the next request.

## 8. Virtual threads (Java 21, not Java 17)

Java 21 adds virtual threads: very cheap threads managed by the JVM. Spring Boot 3.2 and later enables them with `spring.threads.virtual.enabled=true`. Your company uses Java 17, so you only need to know that they exist.

## 9. JVM memory

| Area | Holds | Shared between threads |
|------|-------|------------------------|
| Heap | All objects and arrays | Yes |
| Stack (one per thread) | Method frames: local variables, primitive values, references | No |
| Metaspace | Class metadata | Yes |

```java
void handle() {
    int n = 5;                 // n is on the stack
    User u = new User("a");    // reference u on the stack, User object on the heap
}
```

Errors:

- `StackOverflowError`: recursion that is too deep.
- `OutOfMemoryError: Java heap space`: too many live objects, or a leak (for example, a static map that only grows).

## 10. Garbage collection

The garbage collector (GC) frees objects that no live reference reaches. Java has no manual `free`.

- Generational idea: most objects die young. The heap has a young generation (collected often and fast) and an old generation (collected less often).
- G1 is the default collector since Java 9. ZGC is a low-pause option.
- `System.gc()` is only a hint. Do not call it.
- A memory leak in Java is an object that is still reachable but not needed: static collections, unremoved `ThreadLocal`, unbounded caches, listeners never removed.

## 11. String pool

String literals are stored once in the string pool. `"a" == "a"` is true because both are the same pooled object. `new String("a")` makes a new heap object. This is the reason for the `==` trap in lesson 01.

## Try it

1. Run `CounterService.increment()` from 1,000 tasks on a pool of 8 threads. Print the count. Fix it with `AtomicInteger`.
2. Load a "user" and "orders" in parallel with `CompletableFuture` (use `Thread.sleep` to fake latency) and combine them.
3. Write a recursive method without a base case and read the `StackOverflowError`.
