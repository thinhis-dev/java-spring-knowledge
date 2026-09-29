# Quiz 02: OOP Core

Level: 2 (basic to medium). Covers: Java lesson 02. Time: about 60 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

## Q1

Explain encapsulation in two sentences. Then say why a class with a public getter and setter for every private field is not really encapsulated.

Answer:

## Q2

What is the difference between overloading and overriding? For each one, say at which moment (compile time or runtime) Java chooses the method.

Answer:

## Q3

What does this code print? Explain.

```java
class Animal { String sound() { return "..."; } }
class Dog extends Animal { @Override String sound() { return "Woof"; } }

public class Main {
    static void describe(Animal a) { System.out.println("animal: " + a.sound()); }
    static void describe(Dog d)    { System.out.println("dog: " + d.sound()); }

    public static void main(String[] args) {
        Animal a = new Dog();
        describe(a);
    }
}
```

Answer:

## Q4

Compare an abstract class and an interface in Java 17. Give at least four differences. Which one do you choose by default, and why?

Answer:

## Q5

A class implements two interfaces. Both interfaces have a `default String name()` method. What happens, and how do you fix it?

Answer:

## Q6

State the contract between `equals` and `hashCode`. A class overrides only `equals`. What goes wrong in a `HashSet`?

Answer:

## Q7

Why is this class not immutable? Fix it.

```java
public final class Team {
    private final List<String> members;
    public Team(List<String> members) { this.members = members; }
    public List<String> getMembers() { return members; }
}
```

Answer:

## Q8

Write a Java record `Money` with `amountCents` (long) and `currency` (String). Reject a negative amount and a null currency. How do you read the amount from a `Money` object?

Answer:

## Q9

Why is a record a bad choice for a JPA entity? Give two reasons.

Answer:

## Q10

Write a sealed interface `Shape` with two records, `Circle` and `Square`. Then write a method `double area(Shape s)` with `instanceof` pattern matching.

Answer:

## Q11

What does this code print, and why?

```java
class Parent {
    Parent() { init(); }
    void init() { System.out.println("parent init"); }
}
class Child extends Parent {
    private String name = "child";
    @Override void init() { System.out.println("child init: " + name); }
}
new Child();
```

Answer:

## Q12

What is the difference between a `static` nested class and an inner class? Which one is the default choice, and why?

Answer:

## Q13 (interviewer view)

An intern defines "polymorphism" as "one object can have many forms." Write two follow-up questions that show whether the intern really understands it.

Answer:
