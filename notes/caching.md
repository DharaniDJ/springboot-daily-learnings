# First-Level Caching in JPA

Welcome to **Concept and Coding**! In this project, we explore the concept of **First-Level Caching in JPA**, which is an essential aspect of the JPA architecture. This README provides a detailed walkthrough of the JPA lifecycle, first-level caching, and how it optimizes data access by reducing database queries.

If you're new to JPA or have not seen the previous video on **JPA Architecture and Lifecycle**, we highly recommend revisiting it before diving into this topic.

---

## Table of Contents
1. [Introduction to JPA Lifecycle](#introduction-to-jpa-lifecycle)
2. [What is First-Level Caching?](#what-is-first-level-caching)
3. [How First-Level Caching Works](#how-first-level-caching-works)
4. [Code Walkthrough](#code-walkthrough)
5. [Key Takeaways](#key-takeaways)

---

## Introduction to JPA Lifecycle
The JPA lifecycle consists of multiple states that an entity object transitions through, depending on operations performed. Here's a quick recap:

1. **New (Transient)**: When an entity is created but not yet managed by the EntityManager.
2. **Managed (Persistent)**: When an entity is associated with a persistence context.
3. **Removed**: When an entity is scheduled for deletion but not yet removed from the database.
4. **Detached**: When an entity is no longer associated with the persistence context.

### Example:
```java
UserEntity user = new UserEntity(); // New State
EntityManager em = ...;
em.persist(user); // Managed State
em.remove(user); // Removed State
em.persist(user); // Back to Managed State
em.flush(); // Synchronizes with DB
em.detach(user); // Detached State
```

### Key Point:
The database is only updated when `flush` or `commit` is called. Until then, all changes are held in the **persistence context**, which resides in memory.

---

## What is First-Level Caching?
**First-Level Cache** is a built-in mechanism in JPA where entities are cached within the persistence context of an EntityManager. It:

- Ensures data consistency within the scope of a transaction.
- Reduces database access by serving data directly from the cache when possible.

### Characteristics:
- **Scope**: Limited to a single EntityManager instance.
- **Automatic**: Managed internally by JPA.
- **Isolation**: Each EntityManager has its own persistence context, isolated from others.

---

## How First-Level Caching Works
The persistence context acts as a temporary storage where entities are managed. Here’s how it works step-by-step:

1. **Insert Operation**:
   - When `EntityManager.persist(entity)` is called, the entity is added to the persistence context.
   - It is not immediately inserted into the database; instead, it resides in the cache.

2. **Fetch Operation**:
   - When `EntityManager.find(id)` or `JPARepository.findById(id)` is called, the persistence context is checked first.
   - If the entity exists in the cache, no database query is executed.

3. **Commit or Flush**:
   - When `commit` or `flush` is called, changes in the persistence context are synchronized with the database.

### Example Scenario:
- Insert a user and fetch it within the same persistence context:
  - Insert: `UserRepository.save(user)`
  - Fetch: `UserRepository.findById(userId)`
  - **Result**: Only the insert query is executed; the fetch retrieves data from the cache.

- Fetch a user in a new HTTP request:
  - Fetch: `UserRepository.findById(userId)`
  - **Result**: A new persistence context is created, leading to a fresh SELECT query.

---

## Code Walkthrough
Here’s a step-by-step demonstration:

### Save and Fetch Example:
```java
@RestController
@RequestMapping("/api/jpa")
public class JpaController {

    @Autowired
    private UserRepository userRepository;

    @PostMapping("/save")
    public ResponseEntity<String> saveUser() {
        User user = new User("John Doe");
        userRepository.save(user);
        return ResponseEntity.ok("User saved.");
    }

    @GetMapping("/fetch/{id}")
    public ResponseEntity<User> fetchUser(@PathVariable Long id) {
        User user = userRepository.findById(id).orElse(null);
        return ResponseEntity.ok(user);
    }
}
```

### Observations:
1. On the first request to `/save`, an INSERT query is executed.
2. On the subsequent request to `/fetch/{id}` within the same HTTP request, no SELECT query is executed.
3. On a new HTTP request, a SELECT query is triggered because a new persistence context is created.

### Why?
Each HTTP request generates a new `EntityManager`, which creates an isolated persistence context.

---

## Key Takeaways
- **Single Persistence Context**: One EntityManager equals one persistence context.
- **Automatic Caching**: First-level caching is automatic and doesn’t require additional configuration.
- **Transaction Scope**: The cache is valid only within the transaction scope of the EntityManager.
- **Optimization**: Reduces database hits and improves performance.
- **Isolation**: Persistence contexts are isolated; data in one context is invisible to others.

### Pro Tip:
To leverage caching across requests, consider using a **Second-Level Cache** (e.g., Ehcache, Redis) for advanced scenarios.

