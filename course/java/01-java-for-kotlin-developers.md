# Java 01: Java for Kotlin Developers

You already know the JVM and Spring. This lesson lists the Java rules that surprise a Kotlin developer. Interns write Java, so you must read their code fast and see these traps.

## 1. Everything lives in a class

Java has no top-level functions. Utility code goes into a class with `static` methods.

```java
// Kotlin: fun slugify(s: String) = ...
public final class Strings {
    private Strings() {} // no instances
    public static String slugify(String s) {
        return s.trim().toLowerCase().replace(' ', '-');
    }
}
```

`static` replaces Kotlin's `companion object` and top-level declarations. A static member belongs to the class, not to an object.

## 2. Classes are open by default

| Kotlin | Java |
|--------|------|
| Classes and methods are `final` by default | Classes and methods are open by default |
| `open` to allow inheritance | `final` to forbid inheritance |
| `val` | `final` field or local variable |
| `internal` | No equivalent. Default (no modifier) is package-private |

Java access modifiers, from most open to most closed:

| Modifier | Visible in |
|----------|-----------|
| `public` | Everywhere |
| `protected` | Same package and subclasses |
| (none) package-private | Same package only |
| `private` | Same class only |

## 3. No properties: fields plus getters and setters

```java
public class User {
    private final String email;   // like val
    private String name;          // like var

    public User(String email, String name) {
        this.email = email;
        this.name = name;
    }

    public String getEmail() { return email; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

Frameworks like Jackson and JPA find properties through the `getX` and `setX` names. Kotlin generates these names for you. In Java you write them, or use a `record` (lesson 02), or use Lombok (`@Getter`, `@Data`). Many Java teams use Lombok, so an intern can know `@Data` without knowing what it generates. Ask them.

## 4. Null safety is not in the type system

Every reference type can be `null`. There is no `String?`. The tools are:

- `Objects.requireNonNull(x, "x")` at the start of a method or constructor.
- `Optional<T>` as the return type of a method that can have no result (lesson 05).
- Annotations like `@Nullable` and `@NonNull` for static analysis only.

A `NullPointerException` (NPE) is the most common Java runtime error. Ask interns how they avoid it.

## 5. `==` versus `equals`

In Kotlin, `==` calls `equals`. In Java, `==` compares references for objects.

```java
String a = new String("hi");
String b = new String("hi");
System.out.println(a == b);      // false: two different objects
System.out.println(a.equals(b)); // true: same content

Integer x = 127, y = 127;
System.out.println(x == y);      // true: cached Integer from -128 to 127
Integer p = 128, q = 128;
System.out.println(p == q);      // false: two objects
```

Rule: compare objects with `equals`. Use `==` only for primitives, enums, and a real identity check.

## 6. Primitives and boxing

Java has primitives (`int`, `long`, `boolean`, `double`) and wrapper classes (`Integer`, `Long`, `Boolean`, `Double`). Collections hold only objects, so Java boxes and unboxes automatically. Unboxing a `null` throws an NPE:

```java
Map<String, Integer> stock = new HashMap<>();
int count = stock.get("apple"); // NullPointerException: get returns null
```

## 7. Checked exceptions

Kotlin has no checked exceptions. Java does. A method that throws a checked exception must declare it with `throws`, and the caller must catch it or declare it too. Lesson 06 covers this.

## 8. Strings are immutable

Every change on a `String` makes a new object. In a loop, use `StringBuilder`:

```java
StringBuilder sb = new StringBuilder();
for (String part : parts) sb.append(part).append(',');
String result = sb.toString();
```

Java 15 and later has text blocks for multi-line strings:

```java
String json = """
    {"name": "Ann", "age": 21}
    """;
```

Java has no string templates in Java 17. Use `"Hi " + name` or `String.format("Hi %s", name)` or `"Hi %s".formatted(name)`.

## 9. `var`, `switch`, and `instanceof`

```java
var users = new ArrayList<User>();  // local type inference, Java 10+. Only for local variables.

// Switch expression, Java 14+. Like Kotlin `when`.
String label = switch (status) {
    case ACTIVE -> "Active";
    case BLOCKED, DELETED -> "Inactive";
};

// Pattern matching for instanceof, Java 16+. Like Kotlin smart cast.
if (shape instanceof Circle c) {
    System.out.println(c.radius());
}
```

## 10. No extension functions, no default arguments, no named arguments

- Extension functions: write a static utility method.
- Default arguments: write overloaded constructors or methods, or use the builder pattern (lesson 03).
- Named arguments: use a builder or a parameter object.

## 11. The `main` method

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

## Try it

1. Install JDK 17 (see the README).
2. Write `Main.java` with a `User` class like section 3. Add a `static` counter that counts how many users were created.
3. Reproduce the `Integer` cache surprise from section 5 and the NPE from section 6.
4. Rewrite one small Kotlin class from your work project in Java. Note each place where Java needs more code.
