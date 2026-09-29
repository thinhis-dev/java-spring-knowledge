# Java 06: Exceptions

Kotlin treats every exception as unchecked. Java uses checked exceptions. This affects API design and Spring transaction rollback (Spring lesson 04).

## 1. The hierarchy

```
Throwable
 ├── Error                     JVM problems. Do not catch. OutOfMemoryError, StackOverflowError
 └── Exception                 checked
      ├── IOException, SQLException, ...   checked
      └── RuntimeException     unchecked
           ├── NullPointerException
           ├── IllegalArgumentException
           ├── IllegalStateException
           └── ...
```

| | Checked | Unchecked |
|--|---------|-----------|
| Extends | `Exception` (not `RuntimeException`) | `RuntimeException` |
| Compiler forces handling | Yes: catch or declare `throws` | No |
| Meaning | A recoverable, expected problem outside your control (file missing, network) | A programming bug or a violated rule |

Modern Java and Spring code prefers unchecked exceptions. Spring wraps `SQLException` into unchecked `DataAccessException` for this reason.

## 2. try, catch, finally

```java
try {
    process();
} catch (FileNotFoundException e) {   // more specific first
    log.warn("missing file", e);
} catch (IOException | ParseException e) { // multi-catch
    throw new ImportFailedException("import failed", e); // keep the cause
} finally {
    cleanup(); // runs always, also after return
}
```

Traps:

- A `return` in `finally` hides the exception from `try`. Never return from `finally`.
- Catching `Exception` or `Throwable` everywhere hides bugs.
- An empty catch block swallows the error.
- `e.printStackTrace()` in production code bypasses the logger.

## 3. try-with-resources

Each `AutoCloseable` object in the header closes automatically, in reverse order. This also occurs after an exception.

```java
try (var in = Files.newBufferedReader(path);
     var out = Files.newBufferedWriter(target)) {
    out.write(in.readLine());
} // out closes, then in closes
```

If the body throws and `close()` also throws, the close exception is attached to the main one as a suppressed exception (`e.getSuppressed()`).

## 4. Custom exceptions

```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(long id) {
        super("User " + id + " not found");
    }
}
```

Guidelines:

- Name the problem, not the place: `UserNotFoundException`, not `ServiceException2`.
- If you wrap an exception, always pass the cause: `new XException("msg", cause)`. Without it, you lose the original stack trace.
- Throw early (reject bad input at the start), catch late (at the boundary, for example a Spring `@ControllerAdvice`).
- Do not use exceptions for normal control flow.

## 5. Checked exceptions and lambdas

Standard functional interfaces do not declare checked exceptions, so this does not compile:

```java
paths.stream().map(Files::readString).toList(); // readString throws IOException
```

Fix: catch inside the lambda and wrap in `UncheckedIOException`:

```java
paths.stream().map(p -> {
    try { return Files.readString(p); }
    catch (IOException e) { throw new UncheckedIOException(e); }
}).toList();
```

## 6. Overriding and exceptions

An overriding method cannot throw a broader checked exception than the parent method. It can throw fewer or narrower checked exceptions, and any unchecked exception.

## 7. Exceptions in a REST API

The intern's controller must not return a stack trace to the client. Map domain exceptions to HTTP status codes in one place (Spring lesson 03):

| Exception | HTTP status |
|-----------|-------------|
| Validation failure | 400 Bad Request |
| Not authenticated | 401 Unauthorized |
| Not allowed | 403 Forbidden |
| `UserNotFoundException` | 404 Not Found |
| Duplicate or version conflict | 409 Conflict |
| Unexpected exception | 500 Internal Server Error, with a generic message |

## Try it

1. Write a custom `AutoCloseable` class that prints a line in `close()` and then throws. Use it in a try-with-resources block, throw inside the body, and print `getSuppressed()`.
2. Write a method with `return 1` in `try` and `return 2` in `finally`. Predict and check the result.
3. Wrap a checked exception without the cause, then with the cause. Compare the stack traces.
