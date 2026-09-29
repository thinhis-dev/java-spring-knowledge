# Java 02: OOP Core

OOP (object-oriented programming) organizes code into objects that hold state and behavior. Interviews test four pillars: encapsulation, inheritance, polymorphism, and abstraction. They also test the `equals` and `hashCode` contract, which most interns get wrong.

## 1. Encapsulation

Encapsulation means that an object hides its state and exposes behavior. Other code changes the state only through methods that keep the object valid.

```java
public class BankAccount {
    private long balanceCents; // hidden

    public void deposit(long amountCents) {
        if (amountCents <= 0) throw new IllegalArgumentException("amount must be positive");
        balanceCents += amountCents;
    }

    public void withdraw(long amountCents) {
        if (amountCents > balanceCents) throw new IllegalStateException("insufficient funds");
        balanceCents -= amountCents;
    }

    public long getBalanceCents() { return balanceCents; }
}
```

A class with a public getter and setter for every field is not encapsulated. It is a data bag. Ask an intern: "Why is there no `setBalance`?"

## 2. Constructors and `this`

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public Point() {
        this(0, 0); // calls the other constructor
    }
}
```

If you write no constructor, Java adds an empty no-argument constructor. If you write any constructor, Java does not add it. JPA entities need a no-argument constructor for this reason.

## 3. Inheritance

```java
public class Animal {
    protected final String name;
    public Animal(String name) { this.name = name; }
    public String sound() { return "..."; }
}

public class Dog extends Animal {
    public Dog(String name) {
        super(name); // must be the first statement
    }

    @Override
    public String sound() { return "Woof"; }
}
```

Rules:

- A class extends one class only. It can implement many interfaces.
- `super(...)` calls the parent constructor. If you omit it, Java calls `super()` with no arguments.
- `@Override` is optional, but always use it. If the method does not override anything, the compiler then fails.
- A `final` class cannot be extended. A `final` method cannot be overridden.
- All classes extend `Object`, which gives `equals`, `hashCode`, `toString`, and `getClass`.

## 4. Polymorphism

Polymorphism means that one call does different things based on the real type of the object.

```java
List<Animal> animals = List.of(new Dog("Rex"), new Cat("Tom"));
for (Animal a : animals) {
    System.out.println(a.sound()); // Woof, then Meow
}
```

Two kinds:

| Kind | Also called | Decided at | Example |
|------|-------------|-----------|---------|
| Overriding | Runtime polymorphism, dynamic dispatch | Runtime, by the real object type | `Dog.sound()` replaces `Animal.sound()` |
| Overloading | Compile-time polymorphism | Compile time, by the declared parameter types | `print(int)` and `print(String)` |

Trap: overloading uses the declared type, not the real type.

```java
static void greet(Animal a) { System.out.println("animal"); }
static void greet(Dog d)    { System.out.println("dog"); }

Animal a = new Dog("Rex");
greet(a); // prints "animal": the declared type of a is Animal
```

Static methods are not overridden. A static method with the same signature in a subclass hides the parent method. The call uses the declared type.

## 5. Abstraction: abstract classes and interfaces

```java
public abstract class Shape {
    public abstract double area();          // no body
    public String describe() {              // shared code
        return getClass().getSimpleName() + " with area " + area();
    }
}

public interface Payable {
    long amountCents();                     // abstract by default
    default String currency() { return "USD"; } // Java 8+
    static Payable zero() { return () -> 0; }   // static helper
}
```

| | Abstract class | Interface |
|--|---------------|-----------|
| Multiple inheritance | No, one parent only | Yes, a class implements many |
| State (instance fields) | Yes | No, only `static final` constants |
| Constructors | Yes | No |
| Method bodies | Yes | `default`, `static`, and `private` methods (Java 9+) |
| Use when | Subclasses share state and code, "is-a" relation | You define a capability or contract, "can-do" relation |

Modern answer: prefer interfaces. If subclasses really share state, use an abstract class.

If a class gets the same `default` method from two interfaces, the class must override it and can choose one with `A.super.method()`.

## 6. `equals`, `hashCode`, and `toString`

The contract:

1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true.
2. Equal hash codes do not mean equal objects (collisions are allowed).
3. `equals` must be reflexive, symmetric, transitive, and consistent, and `x.equals(null)` must be false.

If you override `equals` but not `hashCode`, `HashMap` and `HashSet` break:

```java
class Email {
    private final String value;
    Email(String value) { this.value = value; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Email other)) return false;
        return value.equals(other.value);
    }
    // hashCode missing
}

Set<Email> set = new HashSet<>();
set.add(new Email("a@b.com"));
set.contains(new Email("a@b.com")); // false (usually): different hash buckets
```

Fix:

```java
@Override
public int hashCode() { return Objects.hash(value); }
```

A second trap: do not use mutable fields in `hashCode`. If a field changes after you put the object in a `HashSet`, the set cannot find it anymore.

## 7. Immutability

An immutable object cannot change after construction. It is thread-safe and safe to use as a map key. Recipe:

1. Make the class `final`.
2. Make all fields `private final`.
3. Do not write setters.
4. Copy mutable inputs and outputs (defensive copy).

```java
public final class Team {
    private final String name;
    private final List<String> members;

    public Team(String name, List<String> members) {
        this.name = name;
        this.members = List.copyOf(members); // defensive copy, also unmodifiable
    }

    public List<String> members() { return members; }
}
```

## 8. Records (Java 16+)

A record is Java's `data class`. It is final and immutable, and Java generates the constructor, accessors, `equals`, `hashCode`, and `toString`.

```java
public record Money(long amountCents, String currency) {
    public Money {                       // compact constructor for validation
        if (amountCents < 0) throw new IllegalArgumentException("negative");
        Objects.requireNonNull(currency);
    }
    public Money add(Money other) {
        return new Money(amountCents + other.amountCents, currency);
    }
}

Money m = new Money(500, "USD");
m.amountCents(); // accessor is amountCents(), not getAmountCents()
```

Records are good for DTOs and value objects. Records are not good for JPA entities, because JPA needs a no-argument constructor and mutable, non-final classes.

## 9. Sealed types (Java 17)

A sealed type limits which classes can extend it. This is Kotlin's `sealed class`.

```java
public sealed interface PaymentResult permits Success, Failure {}
public record Success(String transactionId) implements PaymentResult {}
public record Failure(String reason) implements PaymentResult {}
```

In Java 17, you use `instanceof` pattern matching on sealed types. Exhaustive pattern matching in `switch` needs Java 21.

## 10. `static` and nested classes

```java
public class Outer {
    static class StaticNested {}   // no link to an Outer instance. Prefer this.
    class Inner {}                 // holds a hidden reference to Outer.this
}
```

Inner (non-static) classes keep the outer object alive. This can cause memory leaks. Default to `static` nested classes.

## Try it

1. Build a `Shape` hierarchy: `sealed interface Shape permits Circle, Rectangle`, both as records, with `area()`.
2. Write a method `static String describe(Shape s)` with `instanceof` patterns.
3. Write the `Email` class without `hashCode`, add it to a `HashSet`, and show that `contains` fails. Then fix it.
4. Make `Team` immutable, then prove that `team.members().add("x")` throws `UnsupportedOperationException`.
