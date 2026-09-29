# Spring 06: Testing

Interns often write either no tests or only `@SpringBootTest` tests that start the whole application. This lesson shows which test type fits which layer.

## 1. The test pyramid in Spring

| Layer | Tool | Starts Spring? | Speed |
|-------|------|----------------|-------|
| Domain and service logic | JUnit 5 + Mockito | No | Milliseconds |
| Controller (HTTP, JSON, validation, status codes) | `@WebMvcTest` + `MockMvc` | Web layer only | Fast |
| Repository (queries, mappings) | `@DataJpaTest` | JPA layer only | Medium |
| Full flow | `@SpringBootTest` + Testcontainers | Everything | Slow |

Most tests go at the bottom. A few full tests cover the main flows.

## 2. Unit test with JUnit 5 and Mockito

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock OrderRepository orderRepository;
    @Mock PaymentClient paymentClient;
    @InjectMocks OrderService orderService;

    @Test
    void createsOrderAndChargesCustomer() {
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        orderService.create(new CreateOrderRequest(1L, List.of(new OrderItemRequest(10L, 2))));

        verify(paymentClient).charge(eq(1L), anyLong());
    }

    @Test
    void throwsWhenOrderIsMissing() {
        when(orderRepository.findById(99L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> orderService.get(99L))
            .isInstanceOf(NotFoundException.class);
    }
}
```

Constructor injection makes this easy: `new OrderService(mockRepo, mockClient)` also works without `@InjectMocks`.

Useful JUnit 5 features: `@DisplayName`, `@ParameterizedTest` with `@CsvSource`, `@Nested`, `@BeforeEach`. AssertJ (`assertThat(...)`) is included in `spring-boot-starter-test` and reads better than `assertEquals`.

## 3. Controller test with `@WebMvcTest`

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired MockMvc mockMvc;
    @MockBean OrderService orderService;   // Boot 3.4+: @MockitoBean

    @Test
    void returns400WhenItemsAreEmpty() throws Exception {
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"customerId": 1, "items": []}
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors.items").exists());
    }

    @Test
    void returns404WhenOrderIsMissing() throws Exception {
        when(orderService.get(99L)).thenThrow(new NotFoundException("Order not found"));

        mockMvc.perform(get("/api/v1/orders/99"))
            .andExpect(status().isNotFound());
    }
}
```

`@WebMvcTest` loads controllers, `@ControllerAdvice`, JSON configuration, and security filters. It does not load services or repositories, so you mock them. If Spring Security is on the classpath, add `@WithMockUser` or import your security configuration.

`@MockBean` is deprecated since Boot 3.4. The replacement is `@MockitoBean` from Spring Framework 6.2.

## 4. Repository test with `@DataJpaTest`

```java
@DataJpaTest
class OrderRepositoryTest {
    @Autowired OrderRepository orderRepository;
    @Autowired TestEntityManager em;

    @Test
    void findsOrdersByStatus() {
        Customer c = em.persist(new Customer("Ann"));
        em.persist(new Order(c, OrderStatus.PAID));
        em.persist(new Order(c, OrderStatus.NEW));
        em.flush();

        assertThat(orderRepository.findByStatus(OrderStatus.PAID)).hasSize(1);
    }
}
```

Each test runs in a transaction that rolls back at the end. By default, it uses an embedded database (H2). H2 does not behave exactly like PostgreSQL. For real SQL behavior, use Testcontainers.

## 5. Integration test with Testcontainers

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class OrderFlowIT {

    @Container
    @ServiceConnection   // Boot 3.1+: sets the datasource URL automatically
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired TestRestTemplate rest;

    @Test
    void createsAndReadsOrder() {
        ResponseEntity<OrderResponse> created =
            rest.postForEntity("/api/v1/orders", validRequest(), OrderResponse.class);
        assertThat(created.getStatusCode()).isEqualTo(HttpStatus.CREATED);
    }
}
```

Testcontainers needs Docker.

## 6. What makes a good test

- The name describes the behavior: `returns404WhenOrderIsMissing`, not `test1`.
- Arrange, act, assert: three clear blocks.
- One behavior per test.
- Tests do not depend on each other or on execution order.
- Do not mock what you do not own in a deep way (for example, a mocked `EntityManager`). Use a slice or integration test for that.
- Do not mock the class under test. Do not mock value objects or DTOs.
- A test that never fails is useless. Break the code once and see the test fail.

## 7. Questions to ask an intern about tests

- What is the difference between a mock and a stub? (A stub returns canned data. A mock also lets you assert how it was called.)
- Why is `@SpringBootTest` for every test a problem? (Slow, and a failure does not show which layer broke.)
- How do you test a method that calls `LocalDateTime.now()`? (Inject a `Clock`.)

## Try it

For your Task API:

1. One Mockito unit test for the service, for the "not found" case.
2. Two `@WebMvcTest` tests: 400 on a blank title, 201 with a `Location` header on success.
3. One `@DataJpaTest` for a custom query.
4. Optional: one Testcontainers test with PostgreSQL. This test needs Docker.
