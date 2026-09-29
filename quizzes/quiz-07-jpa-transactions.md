# Quiz 07: Data JPA and Transactions

Level: 7 (hard). Covers: Spring lesson 04. Time: about 75 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

## Q1

What is the difference between JPA, Hibernate, and Spring Data JPA?

Answer:

## Q2

List five rules for a JPA entity class. Why is Lombok `@Data` risky on an entity?

Answer:

## Q3

Why is `@Enumerated(EnumType.ORDINAL)` dangerous?

Answer:

## Q4

What is the default fetch type for `@ManyToOne`, `@OneToOne`, `@OneToMany`, and `@ManyToMany`? Which defaults do you change?

Answer:

## Q5

Explain the N+1 problem with this code. How many queries run for 200 posts? Give three fixes.

```java
List<Post> posts = postRepository.findAll();
posts.forEach(p -> System.out.println(p.getAuthor().getName()));
```

Answer:

## Q6

Why can `join fetch` on a collection with `Pageable` be a problem? What is a safe pattern?

Answer:

## Q7

What causes `LazyInitializationException`? What does `spring.jpa.open-in-view` do, and why do many teams set it to `false`?

Answer:

## Q8

Does this method update the database? Explain.

```java
@Transactional
public void changeEmail(long id, String email) {
    User u = userRepository.findById(id).orElseThrow();
    u.setEmail(email);
}
```

Answer:

## Q9

After this method, is the order row in the database? Explain the default rollback rule.

```java
@Transactional
public void placeOrder(Order order) throws PaymentException {
    orderRepository.save(order);
    paymentGateway.charge(order); // throws PaymentException (a checked exception)
}
```

Answer:

## Q10

Name three situations in which `@Transactional` has no effect.

Answer:

## Q11

You must save an audit record even after the main transaction rolls back. Which propagation do you use? Write the code.

Answer:

## Q12

The outer method catches an exception from an inner `@Transactional` method with `REQUIRED` propagation, then returns normally. What happens at commit, and why?

Answer:

## Q13

Two admins open the same product and both save a new price. The second save silently overwrites the first one. How do you detect this with JPA, and which HTTP status do you return?

Answer:

## Q14

Compare optimistic and pessimistic locking. Give one use case for each.

Answer:

## Q15

Why must production not use `ddl-auto=update`? What do you use instead?

Answer:

## Q16

In a bidirectional `@OneToMany(mappedBy = "order")` relation, which side is the owner? What happens after you add an item only to `order.getItems()` and do not set `item.setOrder(order)`?

Answer:
