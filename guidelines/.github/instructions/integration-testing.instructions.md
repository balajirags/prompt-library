
---
description: 'Guidelines for building Spring Boot integration tests with Testcontainers and WebTestClient.'
applyTo: '**/*.java'
---


## 🧠 Guidelines

- Annotate test classes with:
  - `@SpringBootTest(webEnvironment = RANDOM_PORT)`
  - `@AutoConfigureWebTestClient`
  - `@Testcontainers`
- Use real Postgres via `Testcontainers`.
- Use `WebTestClient` for HTTP calls.
- Clear DB between tests (`@Sql` or `DbCleaner`).

---

## 💬 Example Prompt

> Create an integration test for `/orders` POST using `WebTestClient` verifying persistence in Postgres via JPA.

---

## ✅ Code Conventions

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@AutoConfigureWebTestClient
class OrderControllerIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @Autowired WebTestClient client;
    @Autowired OrderRepository repo;

    @Test
    void shouldPersistOrder_whenValidRequest() {
        var request = new OrderRequest("sku-123", 2);

        client.post().uri("/orders")
              .bodyValue(request)
              .exchange()
              .expectStatus().isCreated();

        assertThat(repo.findAll()).hasSize(1);
    }
}

```
