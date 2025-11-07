---
description: 'Guidelines for building Spring Data JPA queries with best practices for performance, security, and maintainability.'
applyTo: '**/*.java'
---

## 🧠 Guidelines

- Always use **parameterized** queries (`:param`).
- Explicitly list columns; avoid `SELECT *`.
- Use pagination (`LIMIT`, `OFFSET`) or `Pageable`.
- For write operations, mark `@Modifying(clearAutomatically = true)`.
- Keep transactions at service layer, not repository.

---

## 💬 Example Prompt

> Write a JPA query to fetch top 10 products by sales between two dates, returning a projection DTO.

---

## ✅ Code Conventions

```java
public interface SalesRepository extends JpaRepository<Sale, Long> {

    @Query("""
        SELECT new com.example.dto.TopProductDto(s.productId, SUM(s.amount))
        FROM Sale s
        WHERE s.date BETWEEN :from AND :to
        GROUP BY s.productId
        ORDER BY SUM(s.amount) DESC
    """)
    List<TopProductDto> findTopProducts(@Param("from") LocalDate from, @Param("to") LocalDate to, Pageable pageable);
}

```
