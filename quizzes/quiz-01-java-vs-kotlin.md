# Quiz 01: Java for Kotlin Developers

Level: 1 (basic). Covers: Java lesson 01. Time: about 45 minutes.

Write your answer under each `Answer:` line. Do not open the lessons. Write "I do not know" for an unknown answer.

## Q1

Kotlin classes are `final` by default. What is the default in Java, and which keyword changes it?

Answer:

## Q2

List the four Java access levels from most open to most closed. Which one has no keyword?

Answer:

## Q3

What does this code print? Explain each line.

```java
String a = "hello";
String b = "hello";
String c = new String("hello");
System.out.println(a == b);
System.out.println(a == c);
System.out.println(a.equals(c));
```

Answer:

## Q4

This code throws an exception. Which exception, and why?

```java
Map<String, Integer> stock = new HashMap<>();
int apples = stock.get("apple");
```

Answer:

## Q5

Java has no top-level functions and no `companion object`. Where do you put a utility function like `slugify(String)`, and how do you stop other code from creating an instance of that class?

Answer:

## Q6

Rewrite this Kotlin class in Java without Lombok and without a record. Keep `id` read-only and `name` changeable.

```kotlin
class Product(val id: Long, var name: String)
```

Answer:

## Q7

Java has no default arguments. Name two ways to give a Java class optional constructor parameters.

Answer:

## Q8

Rewrite this Kotlin code in Java 17, with a switch expression.

```kotlin
val label = when (status) {
    Status.ACTIVE -> "Active"
    Status.BLOCKED, Status.DELETED -> "Inactive"
}
```

Answer:

## Q9

What is `var` in Java? Where can you use it and where can you not use it?

Answer:

## Q10

Why is this loop slow for 100,000 items, and how do you fix it?

```java
String result = "";
for (String s : items) {
    result += s + ",";
}
```

Answer:

## Q11

Java has no `String?` type. Name three ways a Java team reduces `NullPointerException` bugs.

Answer:

## Q12 (interviewer view)

An intern says: "In Java, `==` compares values, like in JavaScript." Write the follow-up question that you ask, and the answer that you expect.

Answer:
