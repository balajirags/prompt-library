# instruction.md — Service & Repository Generation Rules (Spring / JPA / Postgres)

> Purpose: Make Copilot generate production‑ready **Repository** + **Service** layers derived from the **current database schema** (entities, migrations, DDL). This file is the **source of truth**.

---

## A) Discover & Trust the Schema (in order)

Copilot must infer the model using the first available source below (stop when confident):

1. **JPA Entities** in `src/main/java/**/(domain|entity)/**` (`@Entity`, `@Table`).
2. **Migrations** in `src/main/resources/db/**` (Liquibase/Flyway; YAML/SQL/XML).
3. **DDL** in `/schema.sql` or `/docs/schema/**`.
4. **ERD/README** under `/docs/**`.

If entities and migrations disagree, **prefer the latest migration**. Align `@Column`, nullability, lengths, uniqueness, indexes accordingly.

---

## B) Repository Layer (Spring Data JPA)

### B1. Structure & Interfaces

* One repository per aggregate root: `XxxRepository extends JpaRepository<Xxx, IdType>, JpaSpecificationExecutor<Xxx>`.
* Return **entities** or **DTO/projections** — never raw `Map` or `Object[]`.
* Use **method names** for simple predicates; use **`Specification`** (or QueryDSL if present) for complex queries.
* Pagination via `Page<T>` + `Pageable` only.

### B2. Mapping & Performance

* For required associations, use `@EntityGraph(attributePaths = {"..."})` to avoid N+1.
* Create **closed projections** for summaries:

  ```java
  interface OrderSummary { UUID getId(); String getOrderNumber(); BigDecimal getTotalAmount(); Instant getCreatedAt(); }
  Page<OrderSummary> findByStatus(OrderStatus status, Pageable pageable);
  ```
* Annotate IDs and columns consistent with DDL (`uuid`, `numeric(19,2)`, `jsonb`, etc.). Add converters when needed.
* Use **`@Version`** for optimistic locking where applicable.

### B3. Example

```java
public interface OrderRepository extends JpaRepository<Order, UUID>, JpaSpecificationExecutor<Order> {
  Optional<Order> findByOrderNumber(String orderNumber);
  boolean existsByOrderNumber(String orderNumber);
  @EntityGraph(attributePaths = {"lines", "customer"})
  Optional<Order> findWithGraphById(UUID id);
}
```

---

## C) Service Layer (Application/Domain)

### C1. Responsibilities

* Transaction boundary (`@Transactional`).
* Orchestration across repositories and domain methods.
* Validation (Jakarta validation + invariants).
* Idempotency & concurrency (optimistic locking retries when safe).
* Mapping: **input DTO → domain → output DTO** (prefer MapStruct if available).
* Publish domain events (post‑commit) if your project uses them.

### C2. Conventions

* Stateless `@Service` beans; **constructor injection** only.
* `@Transactional(readOnly = true)` for queries; explicit `@Transactional` for commands.
* **Never** return JPA entities from controllers; expose DTOs.
* Method names reflect use‑cases: `createOrder`, `addLine`, `cancel`, `markPaid`.

### C3. Example

```java
@Service
@RequiredArgsConstructor
public class OrderService {
  private final OrderRepository orders;
  private final CustomerRepository customers;
  private final OrderMapper mapper; // MapStruct or hand-written
  private final Clock clock;

  @Transactional
  public OrderDto create(CreateOrderCommand cmd) {
    validate(cmd);
    if (orders.existsByOrderNumber(cmd.orderNumber())) {
      throw new ConflictException("Order number exists");
    }
    var customer = customers.findById(cmd.customerId())
        .orElseThrow(() -> new NotFoundException("customer"));

    var order = Order.create(cmd.orderNumber(), customer, cmd.currency(), clock);
    cmd.lines().forEach(l -> order.addLine(l.sku(), l.qty(), l.unitPrice()));

    orders.saveAndFlush(order);
    return mapper.toDto(order);
  }

  @Transactional(readOnly = true)
  public Page<OrderSummaryDto> search(OrderQuery q, Pageable pageable) {
    return orders.findAll(OrderSpecifications.from(q), pageable)
                 .map(mapper::toSummary);
  }
}
```

---

## D) Specifications (Composition over query strings)

Provide a single factory to compose filters safely:

```java
public final class OrderSpecifications {
  public static Specification<Order> from(OrderQuery q) {
    return Specification.where(hasStatus(q.status()))
        .and(createdBetween(q.from(), q.to()))
        .and(customerIdEquals(q.customerId()))
        .and(freeText(q.q()));
  }
  // Each method returns null-safe Specification segments
}
```

---

## E) DTOs & Mapping

* DTOs are **immutable** (Java 17 `record` or Lombok `@Value`).
* Decorate with Jakarta validation (`@NotBlank`, `@Size`, `@Positive`…).
* MapStruct: `@Mapper(componentModel = "spring")`, split toDto/toSummary.
* Hide internal IDs in API unless required; expose business keys where appropriate.

**Command example**

```java
public record CreateOrderCommand(
  @NotBlank String orderNumber,
  @NotNull UUID customerId,
  @NotNull Currency currency,
  @Size(min = 1) List<CreateOrderLine> lines
) {}
```

---

## F) Concurrency & Idempotency

* Use `@Version` for aggregates; on `OptimisticLockingFailureException` retry a small, bounded number of times when safe.
* For create operations, enforce uniqueness via `existsBy…` checks and DB unique constraints (with conflict handling).

```java
@Retryable(retryFor = OptimisticLockingFailureException.class, maxAttempts = 3, backoff = @Backoff(delay = 50, multiplier = 2))
@Transactional
public void addLine(UUID orderId, AddLine cmd) { /* ... */ }
```

---

## G) Observability & Logging

* **INFO**: business events (created/updated) with key identifiers.
* **WARN**: degraded but handled (fallback, retry, timeout with recovery).
* **ERROR**: operation failed; pair with exception.
* Include correlation/trace IDs via MDC; never log PII or secrets.

---

## H) What NOT to do

* No raw `Map`/`Object[]` from repositories.
* Don’t expose entities in controllers.
* Don’t perform network IO in repositories (DB only).
* Don’t scatter transaction boundaries across layers.

---

## I) Postgres / Type Mapping Notes

* Map `uuid` → `java.util.UUID`.
* Map `jsonb` → `String` or custom type with `@Converter`.
* Use `Instant`/`OffsetDateTime` for timestamps; store in UTC.

---

# prompts.md — Reusable Prompts for Copilot (Service + Repository)

> Use these snippets at the **top of the file you’re editing** (or in a scratch buffer in the same module) before asking Copilot to generate code. Replace ⟪placeholders⟫.

---

## Prompt 1 — Generate Repository + Projections

```
GOAL: Create ⟪AggregateName⟫Repository for entity ⟪com.example.domain.⟪Aggregate⟫⟫ with id ⟪UUID/Long⟫.
CONTEXT: Follow /instruction.md (Repository rules). Prefer latest migrations if entities disagree with DDL.
STACK: Spring Boot ⟪version⟫, Spring Data JPA, Postgres. QueryDSL ⟪yes/no⟫.

REQUIREMENTS:
- extends JpaRepository<⟪Aggregate⟫, ⟪IdType⟫> and JpaSpecificationExecutor<⟪Aggregate⟫>
- finders: findBy⟪BusinessKey⟫(...), existsBy⟪BusinessKey⟫(...)
- paged query by status/date using Page<T> + Pageable
- @EntityGraph for required associations to avoid N+1
- projection interface ⟪AggregateName⟫Summary { … } for list views
- column annotations aligned with migrations (nullable, length, unique)

GENERATE NOW.
```

---

## Prompt 2 — Generate Specifications Factory

```
GOAL: Create ⟪AggregateName⟫Specifications for composing filters into Specification<⟪Aggregate⟫>.
CONTEXT: Follow /instruction.md (Specifications rules). Use null‑safe segments.
FILTERS: status, createdAt between ⟪from⟫..⟪to⟫, customerId, freeText on ⟪fields⟫.
OUTPUT: final class with static from(Query) and private helpers.
GENERATE NOW.
```

---

## Prompt 3 — DTOs & Mapper

```
GOAL: Create immutable DTOs and a mapper for ⟪AggregateName⟫.
CONTEXT: Java 17 records ⟪yes/no⟫, MapStruct ⟪yes/no⟫.
REQUIRED:
- Create⟪AggregateName⟫Command (Jakarta validation)
- Update⟪AggregateName⟫Command
- ⟪AggregateName⟫Dto and ⟪AggregateName⟫SummaryDto
- If MapStruct: @Mapper(componentModel="spring") with toDto/toSummary.
GENERATE NOW.
```

---

## Prompt 4 — Service Layer (Transactions, Validation, Conflicts)

```
GOAL: Implement ⟪AggregateName⟫Service orchestrating repository, validation, and mapping.
CONTEXT: Follow /instruction.md (Service rules). Do not return entities to controllers.
METHODS:
- create(Create⟪AggregateName⟫Command) → ⟪AggregateName⟫Dto
- getById(⟪IdType⟫) → ⟪AggregateName⟫Dto (NotFound on missing)
- search(⟪AggregateName⟫Query, Pageable) → Page<⟪AggregateName⟫SummaryDto>
- update(⟪IdType⟫, Update⟪AggregateName⟫Command) with optimistic locking if entity has @Version
- domain ops: ⟪e.g., addLine / cancel⟫ with invariants

RULES:
- @Transactional(readOnly=true) for queries; explicit @Transactional for commands
- existsBy⟪BusinessKey⟫ to enforce uniqueness + DB unique constraint handling
- Parameterized logging with correlation IDs (assume MDC)
GENERATE NOW.
```

---

## Prompt 5 — Controller Scaffold (Optional)


