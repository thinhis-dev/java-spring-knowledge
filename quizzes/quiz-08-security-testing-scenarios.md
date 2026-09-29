# Quiz 08: Security, Testing, and Scenarios

Level: 8 (advanced). Covers: Spring lessons 05 and 06, plus all earlier lessons. Time: about 90 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

## Part A: Security

### Q1

What is the difference between authentication and authorization? Which HTTP status belongs to each failure?

Answer:

### Q2

Describe the path of a request through the Spring Security filter chain to your controller. Where is the current user stored?

Answer:

### Q3

Write a Spring Security 6 `SecurityFilterChain` for a stateless JWT API: `/api/v1/auth/**` is public, `DELETE` needs `ADMIN`, everything else needs authentication, and CORS allows `http://localhost:3000`.

Answer:

### Q4

When is it safe to disable CSRF protection? When must you keep it?

Answer:

### Q5

A teammate says: "JWT is encrypted, so we put the user's email and address in it." What is wrong? Name two other JWT facts that an intern must know.

Answer:

### Q6

A user changes `GET /api/v1/invoices/1001` to `/1002` and sees another customer's invoice. What is this vulnerability called? Fix it in the service code.

Answer:

### Q7

Compare `localStorage` and an `HttpOnly` cookie for storing a token in a React app.

Answer:

## Part B: Testing

### Q8

Which test type do you use for each case, and why?

1. The discount calculation in `PricingService`.
2. The controller returns 400 for a blank name.
3. A custom `@Query` with a join.
4. The full order flow with PostgreSQL.

Answer:

### Q9

`OrderService.get(99L)` throws `NotFoundException`. Write a `@WebMvcTest` test that expects 404 from `GET /api/v1/orders/99`.

Answer:

### Q10

How do you test a method that uses `LocalDateTime.now()`?

Answer:

### Q11

What is the difference between a mock and a stub? What is wrong with using `@SpringBootTest` for every test?

Answer:

## Part C: Scenarios

### Q12

Review this service method. List every problem that you find, and rewrite it.

```java
@Service
public class TransferService {
    @Autowired AccountRepository repo;

    public void transfer(Long from, Long to, double amount) {
        Account a = repo.findById(from).get();
        Account b = repo.findById(to).get();
        a.setBalance(a.getBalance() - amount);
        b.setBalance(b.getBalance() + amount);
        repo.save(a);
        repo.save(b);
    }
}
```

Answer:

### Q13

The `GET /api/v1/orders` endpoint takes 4 seconds in production. Describe your investigation steps, in order, and the likely causes from this course.

Answer:

### Q14

Design the API for a "task board" feature for your React app: tasks have a title, status (TODO, DOING, DONE), assignee, and comments. Give the endpoints, methods, status codes, one request DTO, one response DTO, and the pagination approach.

Answer:

## Part D: Interviewer skills

### Q15

For each topic, write one intern question, the minimum passing answer, and one red-flag answer:

1. `equals` and `hashCode`
2. Dependency injection
3. HTTP status codes
4. `@Transactional`

Answer:

### Q16

An intern answers every theory question well, but cannot write a simple controller in a live exercise. Another intern is weak on theory, but writes clean, tested code. Which signals do you weigh, and how do you decide?

Answer:
