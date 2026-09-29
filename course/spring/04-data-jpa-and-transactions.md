# Spring 04: Data JPA and Transactions

Most production bugs in Spring apps come from JPA: N+1 queries, lazy loading errors, and misunderstood transactions. Interns rarely know these. You must know them well to review their pull requests.

## 1. Vocabulary

| Term | Meaning |
|------|---------|
| JPA | Jakarta Persistence API: the standard interfaces (`jakarta.persistence`) |
| Hibernate | The JPA implementation that Spring Boot uses |
| Spring Data JPA | Repositories on top of JPA, so you write fewer queries |
| Entity | A class mapped to a table |
| Persistence context | The first-level cache of managed entities inside one transaction |

## 2. An entity

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)     // never ORDINAL: reordering the enum corrupts data
    @Column(nullable = false)
    private OrderStatus status;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "customer_id")
    private Customer customer;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    @Version
    private Long version;            // optimistic locking

    protected Order() {}             // required by JPA

    public void addItem(OrderItem item) {  // keep both sides in sync
        items.add(item);
        item.setOrder(this);
    }
}
```

Entity rules:

- A no-argument constructor (can be `protected`).
- Not `final`, so that Hibernate can create lazy proxies.
- Not a record.
- `equals` and `hashCode`: do not use all fields and do not use Lombok `@Data`. Use a business key, or the ID with care for new entities that have no ID yet.
- Do not use Lombok `@ToString` with relations. It triggers lazy loading and infinite recursion.

## 3. Relations and fetch types

| Relation | Default fetch | Recommended |
|----------|---------------|-------------|
| `@ManyToOne` | EAGER | Set LAZY |
| `@OneToOne` | EAGER | Set LAZY |
| `@OneToMany` | LAZY | Keep LAZY |
| `@ManyToMany` | LAZY | Keep LAZY. Often better as an explicit join entity |

In a bidirectional relation, the side with `mappedBy` is the inverse side. The other side (with the foreign key) is the owner. Only changes on the owner side are saved.

## 4. Repositories

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    List<Order> findByStatusAndCreatedAtAfter(OrderStatus status, Instant after); // derived query

    @Query("select o from Order o where o.customer.id = :customerId")
    Page<Order> findForCustomer(@Param("customerId") Long customerId, Pageable pageable);

    boolean existsByCustomerIdAndStatus(Long customerId, OrderStatus status);
}
```

For DTO projections that skip the entity:

```java
public interface OrderSummary { Long getId(); OrderStatus getStatus(); }
List<OrderSummary> findByCustomerId(Long customerId);
```

## 5. The N+1 problem

```java
List<Order> orders = orderRepository.findAll();              // 1 query
for (Order o : orders) {
    System.out.println(o.getCustomer().getName());           // 1 query per order
}
// 100 orders = 101 queries
```

Fixes:

```java
@Query("select o from Order o join fetch o.customer")         // one join query
List<Order> findAllWithCustomer();

@EntityGraph(attributePaths = {"customer", "items"})
List<Order> findByStatus(OrderStatus status);
```

Other options: `@BatchSize` or `spring.jpa.properties.hibernate.default_batch_fetch_size=50`, which loads lazy relations in batches with `IN (...)`. A DTO projection query is also a fix.

Trap: `join fetch` on a collection with pagination makes Hibernate paginate in memory (warning `HHH90003004`). Page the IDs first, then fetch the details.

To see the queries in development: `spring.jpa.show-sql=true`, or log `org.hibernate.SQL` at debug level.

## 6. Lazy loading and `open-in-view`

`LazyInitializationException` is the result of a read of a lazy relation after the persistence context closed. Example: the controller maps an entity to JSON after the service transaction ended.

Spring Boot enables `spring.jpa.open-in-view=true` by default and logs a warning. This keeps the persistence context open during the whole HTTP request. It hides the error but runs hidden queries in the web layer and holds a database connection longer. Many teams set it to `false` and load what they need in the service, then map to DTOs there.

## 7. Entity states and dirty checking

| State | Meaning |
|-------|---------|
| Transient | `new Order()`, not known to JPA |
| Managed | Loaded or saved inside the current persistence context |
| Detached | Was managed, but the context closed |
| Removed | Scheduled for delete |

Dirty checking: inside a transaction, a change to a managed entity is saved at commit. You do not need to call `save`.

```java
@Transactional
public void rename(long id, String name) {
    Customer c = customerRepository.findById(id).orElseThrow();
    c.setName(name); // UPDATE at commit, no save() call needed
}
```

## 8. `@Transactional`

```java
@Service
public class OrderService {
    @Transactional
    public OrderResponse create(CreateOrderRequest req) { /* several writes: all or nothing */ }

    @Transactional(readOnly = true)
    public OrderResponse get(long id) { /* readOnly skips dirty checking, a small optimization */ }
}
```

Put it on service methods, not on controllers or repositories.

Rollback rules: by default, Spring rolls back on unchecked exceptions (`RuntimeException`, `Error`) only. A checked exception commits the transaction.

```java
@Transactional(rollbackFor = Exception.class) // roll back on checked exceptions too
```

It does not work (Spring lesson 01, section 7):

- On `private` methods.
- On self-invocation (`this.method()`).
- If you catch the exception inside the method and do not rethrow it.

## 9. Propagation

| Propagation | Behavior |
|-------------|----------|
| `REQUIRED` (default) | Join the current transaction, or start one |
| `REQUIRES_NEW` | Suspend the current transaction and start a new one. Example: an audit log that must stay in the database also after a failure of the main work |
| `MANDATORY` | Throw an exception, because no transaction exists |
| `SUPPORTS` | Join an existing transaction. With no transaction, run without one |
| `NOT_SUPPORTED` | Suspend any transaction |
| `NESTED` | Savepoint inside the current transaction (JDBC only) |

Trap with `REQUIRED`: an inner method throws a runtime exception, and the outer method catches it. The whole transaction is still marked rollback-only. The commit then fails with `UnexpectedRollbackException`.

## 10. Isolation and locking

Isolation levels, from weak to strong: `READ_UNCOMMITTED`, `READ_COMMITTED` (PostgreSQL default), `REPEATABLE_READ` (MySQL InnoDB default), `SERIALIZABLE`.

Two users edit the same order:

- Optimistic locking: a `@Version` column. The second update fails with `OptimisticLockException`. Return 409 Conflict. Good for rare conflicts, which is the normal web case.
- Pessimistic locking: `@Lock(LockModeType.PESSIMISTIC_WRITE)` runs `SELECT ... FOR UPDATE`. Good for hot rows, for example stock counters.

## 11. Migrations

Never use `spring.jpa.hibernate.ddl-auto=update` in production. Use Flyway or Liquibase with versioned SQL files (`V1__create_orders.sql`), and set `ddl-auto=validate` or `none`.

## Try it

1. Create `Customer` and `Order` entities with H2. Insert 10 orders for 3 customers.
2. Turn on SQL logging, load all orders, and print each customer name. Count the queries.
3. Fix it with `join fetch`, then with `@EntityGraph`. Count again.
4. Set `open-in-view=false`, return an entity from a controller, and read the `LazyInitializationException`.
5. Throw a checked exception from a `@Transactional` method after a save. Show that the row stays. Then add `rollbackFor`.
