# Quiz 06: REST API Best Practice

Level: 6 (medium to hard). Covers: Spring lesson 03. Time: about 75 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

## Q1

Fix these endpoints to follow REST conventions:

1. `GET /getAllUsers`
2. `POST /users/delete?id=5`
3. `GET /user/5/getOrders`
4. `POST /updateUserEmail`
5. `GET /orders/paid`

Answer:

## Q2

Explain "safe" and "idempotent". Mark each of GET, POST, PUT, PATCH, and DELETE.

Answer:

## Q3

A mobile client sends `POST /payments`, gets a timeout, and retries. How do you stop a double charge?

Answer:

## Q4

Give the status code for each case:

1. A new order is created.
2. The JSON body is broken.
3. No token is sent.
4. A valid user tries to delete another user's order.
5. The order ID does not exist.
6. The email is already registered.
7. A resource is deleted.
8. A `NullPointerException` happens in the service.

Answer:

## Q5

Find at least five problems in this controller.

```java
@RestController
public class UserController {
    @Autowired
    private UserRepository userRepository;

    @PostMapping("/createUser")
    public User create(@RequestBody User user) {
        if (user.getEmail() == null) {
            return null;
        }
        return userRepository.save(user);
    }

    @GetMapping("/users")
    public List<User> all() {
        return userRepository.findAll();
    }
}
```

Answer:

## Q6

Give four reasons not to return JPA entities from a controller.

Answer:

## Q7

Write `CreateUserRequest` and `UserResponse` records. The request has `email` (required, email format), `name` (required, max 50), and `age` (at least 16). The response has no password.

Answer:

## Q8

The request DTO has constraint annotations, but invalid data still reaches the service. What is the most likely cause?

Answer:

## Q9

What is the difference between `@NotNull`, `@NotEmpty`, and `@NotBlank` for the values `null`, `""`, and `"  "`?

Answer:

## Q10

Write a `@RestControllerAdvice` that returns Problem Details for:

1. `NotFoundException` as 404.
2. Validation errors as 400, with a map of field to message.
3. Any other exception as 500, with a generic message, and a log of the real error.

Answer:

## Q11

Your React table shows 50,000 orders and the page freezes. Design the API change. Name two pagination styles and the trade-off between them.

Answer:

## Q12

Compare three API versioning styles. Which change is breaking and which is not: add an optional field, rename a field, remove an endpoint, add a new endpoint?

Answer:

## Q13

The React app on `localhost:3000` gets "blocked by CORS policy" on `localhost:8080`. Explain what the browser does and where you fix it in a Spring Security app.

Answer:

## Q14

Why must an API not use `double` for money? How do you send dates and times in JSON?

Answer:

## Q15 (interviewer view)

Write three questions that you ask an intern about REST APIs, from easy to hard. For each, write the minimum answer that passes.

Answer:
