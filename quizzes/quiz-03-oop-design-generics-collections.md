# Quiz 03: OOP Design, Generics, and Collections

Level: 3 (medium). Covers: Java lessons 03 and 04. Time: about 60 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

## Q1

Name the five SOLID principles. For each one, write one sentence in your own words.

Answer:

## Q2

Which SOLID principle does this code break? Refactor it.

```java
public class ShippingCost {
    public long calculate(Order order) {
        if (order.carrier().equals("DHL")) return 500;
        if (order.carrier().equals("UPS")) return 700;
        if (order.carrier().equals("FEDEX")) return 800;
        throw new IllegalArgumentException("unknown carrier");
    }
}
```

Answer:

## Q3

Explain "composition over inheritance" with a short example. When is inheritance still the right choice?

Answer:

## Q4

Why does `Square extends Rectangle` break the Liskov substitution principle?

Answer:

## Q5

Which principle does this code break? Why is it hard to test?

```java
public class ReportService {
    private final PdfRenderer renderer = new PdfRenderer();
    private final SmtpMailer mailer = new SmtpMailer("smtp.example.com");
}
```

Answer:

## Q6

Choose a collection for each need and explain in one line:

1. Keep unique user IDs, with fast `contains`.
2. Keep unique tags in the order the user added them.
3. Keep scores sorted, and find the next score above 80.
4. Use a stack.
5. Count words from many threads at the same time.

Answer:

## Q7

Explain step by step what `HashMap.put(key, value)` does. What changed in Java 8 for buckets with many collisions?

Answer:

## Q8

What happens here, and why?

```java
class Person {
    String name;
    Person(String name) { this.name = name; }
    @Override public boolean equals(Object o) { return o instanceof Person p && p.name.equals(name); }
    @Override public int hashCode() { return name.hashCode(); }
}

Map<Person, Integer> ages = new HashMap<>();
Person ann = new Person("Ann");
ages.put(ann, 30);
ann.name = "Anna";
System.out.println(ages.get(ann));
System.out.println(ages.get(new Person("Ann")));
```

Answer:

## Q9

What happens here? Give the correct code.

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4));
for (Integer n : nums) {
    if (n % 2 == 0) nums.remove(n);
}
```

Answer:

## Q10

What is the difference between `List.of(...)`, `Arrays.asList(...)`, and `new ArrayList<>(...)` for `add`, `set`, and `null` values?

Answer:

## Q11

Explain type erasure. Give two things that you cannot do in Java because of it.

Answer:

## Q12

Why does the first line not compile? Explain PECS and fix the method signature of `printAll` and `addNumbers`.

```java
List<Number> numbers = new ArrayList<Integer>();

static void printAll(List<Number> list) { list.forEach(System.out::println); }
static void addNumbers(List<Integer> list) { list.add(1); list.add(2); }
```

Answer:

## Q13

Why is `return a.score() - b.score();` a bad `compare` implementation? Write a better comparator that sorts by score descending, then by name.

Answer:

## Q14

Write a builder for an `Email` class with a required `to`, and an optional `subject` (default `""`) and `cc` list (default empty).

Answer:
