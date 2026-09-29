# Spring 03: REST API Best Practice

This is the most important lesson for intern interviews. Your frontend background helps: you know what a bad API feels like from the React side.

## 1. Resource design

| Rule | Good | Bad |
|------|------|-----|
| Nouns, plural | `GET /orders` | `GET /getOrders` |
| Hierarchy for ownership | `GET /users/42/orders` | `GET /orders?userId=42` for a strict child resource |
| Query parameters for filter, sort, and page | `GET /orders?status=PAID&sort=createdAt,desc&page=0&size=20` | `GET /orders/paid` |
| Lowercase, kebab-case | `/order-items` | `/OrderItems` |
| Actions that are not CRUD | `POST /orders/42/cancel` | `GET /cancelOrder?id=42` |

## 2. HTTP methods

| Method | Use | Safe | Idempotent |
|--------|-----|------|------------|
| GET | Read | Yes | Yes |
| POST | Create, or run an action | No | No |
| PUT | Replace the whole resource | No | Yes |
| PATCH | Change part of the resource | No | Not guaranteed |
| DELETE | Remove | No | Yes |

Safe means no change on the server. Idempotent means that the same request sent many times has the same effect as one time. A client can safely retry an idempotent request after a timeout.

To make `POST /payments` safe to retry, the client sends an `Idempotency-Key` header. The server stores the key and returns the first result for a repeated key.

## 3. Status codes

| Code | Use |
|------|-----|
| 200 OK | Successful GET, PUT, PATCH with a body |
| 201 Created | Successful POST that created a resource. Add a `Location` header |
| 204 No Content | Successful DELETE, or success without a body |
| 400 Bad Request | Malformed JSON or failed validation |
| 401 Unauthorized | Not authenticated (no or bad token) |
| 403 Forbidden | Authenticated, but not allowed |
| 404 Not Found | The resource does not exist |
| 409 Conflict | Duplicate, or a version conflict |
| 422 Unprocessable Content | Valid syntax, but a business rule fails. Some teams use 400 for this |
| 500 Internal Server Error | A bug. Never expose the stack trace |

Red flag in an intern: `200 OK` with `{"success": false}` in the body.

## 4. A controller

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) { this.orderService = orderService; }

    @GetMapping("/{id}")
    public OrderResponse get(@PathVariable long id) {
        return orderService.get(id);
    }

    @GetMapping
    public Page<OrderResponse> list(@RequestParam(required = false) OrderStatus status,
                                    Pageable pageable) {
        return orderService.list(status, pageable);
    }

    @PostMapping
    public ResponseEntity<OrderResponse> create(@Valid @RequestBody CreateOrderRequest req) {
        OrderResponse created = orderService.create(req);
        URI location = URI.create("/api/v1/orders/" + created.id());
        return ResponseEntity.created(location).body(created);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable long id) {
        orderService.delete(id);
    }
}
```

| Annotation | Reads from |
|-----------|-----------|
| `@PathVariable` | The URL path: `/orders/{id}` |
| `@RequestParam` | The query string: `?status=PAID` |
| `@RequestBody` | The JSON body, converted by Jackson |
| `@RequestHeader` | A header |

`@RestController` is `@Controller` plus `@ResponseBody`. The return value is written as JSON, not resolved as a view name.

## 5. DTOs, not entities

A DTO (data transfer object) is a class that only carries data across the API boundary. Do not return JPA entities from controllers:

- The entity exposes internal fields (password hash, internal flags).
- Lazy relations cause `LazyInitializationException` or extra queries during JSON serialization.
- Bidirectional relations cause infinite JSON recursion.
- A database change breaks the API contract, and the React client breaks with it.
- With an entity as the request body, a client can set fields that it must not set (mass assignment), for example `role` or `id`.

Use records:

```java
public record CreateOrderRequest(
        @NotNull Long customerId,
        @NotEmpty List<@Valid OrderItemRequest> items) {}

public record OrderItemRequest(@NotNull Long productId, @Min(1) int quantity) {}

public record OrderResponse(long id, String status, long totalCents, Instant createdAt) {
    static OrderResponse from(Order o) {
        return new OrderResponse(o.getId(), o.getStatus().name(), o.getTotalCents(), o.getCreatedAt());
    }
}
```

Map by hand for small projects, or use MapStruct for large ones.

## 6. Validation

Add `spring-boot-starter-validation`. Common constraints: `@NotNull`, `@NotBlank`, `@NotEmpty`, `@Size`, `@Min`, `@Max`, `@Email`, `@Pattern`, `@Past`, `@Future`.

- `@Valid` on `@RequestBody` triggers validation. Without it, the annotations do nothing.
- For nested objects and list items, add `@Valid` on the field too.
- For `@PathVariable` and `@RequestParam` constraints, Boot 3.1 and later (Spring 6.1) validates them on the controller method without `@Validated` on the class.
- Validation at the API edge covers input format. Business rules (for example, "stock is available") belong in the service.

Difference: `@NotNull` accepts `""`. `@NotEmpty` rejects `""` but accepts `" "`. `@NotBlank` rejects both (strings only).

## 7. Error handling in one place

Spring 6 supports RFC 9457 Problem Details (earlier RFC 7807), a standard JSON error format. Use `@RestControllerAdvice`:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ProblemDetail notFound(NotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail invalid(MethodArgumentNotValidException ex) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Validation failed");
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        pd.setProperty("errors", errors);
        return pd;
    }

    @ExceptionHandler(Exception.class)
    public ProblemDetail unexpected(Exception ex) {
        log.error("Unexpected error", ex);  // full detail in the log
        return ProblemDetail.forStatusAndDetail(HttpStatus.INTERNAL_SERVER_ERROR, "Unexpected error"); // generic for the client
    }
}
```

Response:

```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed",
  "instance": "/api/v1/orders",
  "errors": { "items": "must not be empty" }
}
```

To turn on Problem Details for Spring's built-in exceptions too, set `spring.mvc.problemdetails.enabled=true`. A consistent error format lets the React side show field errors next to form inputs.

## 8. Pagination, sorting, and filtering

Never return an unbounded list. Spring Data gives `Pageable`:

```
GET /api/v1/orders?page=0&size=20&sort=createdAt,desc
```

Limit the maximum page size (`spring.data.web.pageable.max-page-size=100`). Offset pagination gets slow on very large tables. Cursor (keyset) pagination (`?after=<lastId>`) scales better for infinite scroll.

Note: returning `Page<T>` directly gives a JSON shape that Spring does not promise to keep stable. Spring Data 3.3 and later logs a warning about it. Return your own page DTO, or enable `@EnableSpringDataWebSupport(pageSerializationMode = VIA_DTO)`.

## 9. Versioning

| Style | Example | Notes |
|-------|---------|-------|
| URL path | `/api/v1/orders` | Most common, visible, easy to route |
| Header | `X-API-Version: 2` | Clean URLs, harder to test in a browser |
| Media type | `Accept: application/vnd.shop.v2+json` | Most "correct", least common |

Add a new version only for breaking changes. Adding an optional field is not a breaking change. Removing or renaming a field is.

## 10. Other practices

- Use `Instant` or `OffsetDateTime` and ISO 8601 in JSON (`2026-09-29T10:00:00Z`). Do not send local time without a zone.
- Money: use integer cents or `BigDecimal`, never `double`.
- Use `Long` or UUID IDs. Sequential IDs let a user guess other resources, so the server must make sure that the user owns the resource (lesson 05).
- CORS: the browser blocks a React app on `localhost:3000` that calls `localhost:8080` unless the server allows it. Configure CORS in Spring Security (lesson 05), not with `@CrossOrigin` on every controller.
- Use OpenAPI (`springdoc-openapi`) to generate API documentation and TypeScript types for the frontend.
- Rate-limit public endpoints.
- Keep controllers thin: about 5 lines per method.

## Try it

Build a small `Task` API (in-memory map or H2):

1. `GET /api/v1/tasks` with pagination, `GET /api/v1/tasks/{id}`, `POST`, `PUT`, `DELETE`.
2. Use request and response records. Validate `title` with `@NotBlank @Size(max = 100)`.
3. Return 201 with `Location` for POST and 204 for DELETE.
4. Add a `GlobalExceptionHandler` with Problem Details for 404, 400, and 500.
5. Call it from `curl` or from a small React page. Trigger each error on purpose.
