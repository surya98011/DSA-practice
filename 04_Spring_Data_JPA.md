# Part 4: Data + JPA & Hibernate internals

This is a heavily tested section for senior Java Developers. Knowing how the ORM operates beneath the abstractions is crucial to prevent N+1 problems and database deadlocks.

---

### JPA vs Hibernate vs Spring Data JPA

- **JPA (Java Persistence API)**: It is just a specification/interface (`javax.persistence.*`). It dictates how Object-Relational Mapping (ORM) *should* work.
- **Hibernate**: It is an implementation of JPA. It does the heavy lifting of converting POJOs to SQL queries.
- **Spring Data JPA**: It is an abstraction layer *on top* of JPA to reduce boilerplate code. It provides `Repository` interfaces (`save()`, `findById()`) so you don't have to manually manage `EntityManager`.

**CrudRepository vs JpaRepository?**
- `CrudRepository` provides core abstract methods: save, findById, delete.
- `JpaRepository` extends `CrudRepository` and `PagingAndSortingRepository`. It supports sorting/pagination and provides JPA-specific features like `saveAndFlush()` and `deleteInBatch()`.

**Q: What is @GeneratedValue?**
Dictates how the database generates primary keys.
- `GenerationType.IDENTITY`: Relies on an auto-incremented database column (MySQL).
- `GenerationType.SEQUENCE`: Uses a database sequence object (PostgreSQL/Oracle). (Generally preferred for batch inserts over IDENTITY).
- `GenerationType.UUID`: Creates universally unique identifiers.

---

### Transactions & Locking (Q65, 66, 67, 68)

**Q: What is @Transactional and how it works internally?**
Spring uses **AOP Proxying**. When you call an `@Transactional` method, you aren't calling the actual class. You are calling a dynamically generated Proxy class that:
1. Opens a Transaction (via `TransactionManager`).
2. Tries to execute your actual method.
3. If successful, it commits.
4. If a `RuntimeException` or `Error` occurs, it automatically rolls back. (Note: Unchecked exceptions trigger rollback, Checked exceptions do NOT unless explicitly specified).

**Q: Transaction Propagation Types?**
- `REQUIRED` (Default): If a transaction exists, join it. Else, create a new one.
- `REQUIRES_NEW`: Suspend the current transaction and create a completely new, independent transaction.
- `SUPPORTS`: Execute within a transaction if one exists. Else, execute non-transactionally.

**Q: Transaction Isolation Levels?**
- `READ_UNCOMMITTED`: Can read uncommitted data (Dirty Reads).
- `READ_COMMITTED`: Solves dirty reads. Can still encounter Non-Repeatable Reads. (PostgreSQL Default).
- `REPEATABLE_READ`: Solves Non-Repeatable reads. Can still encounter Phantom Reads. (MySQL Default).
- `SERIALIZABLE`: Fully locked, sequential execution. Solves all problems but terrible performance.

**Q: Optimistic vs Pessimistic locking? What is @Version?**
- **Optimistic Locking**: Assumes simultaneous updates are rare. Uses an `@Version` column in JPA. Before updating, JPA checks if the version in the DB matches the version in memory. If not, throws `OptimisticLockException`. (No database locks held).
- **Pessimistic Locking**: `SELECT ... FOR UPDATE`. Used when concurrent collisions are highly likely (e.g., Financial debiting). Secures an exclusive lock at the database level, preventing other threads from reading/writing until the transaction commits.

---

### Performance Optimization: N+1 & Lazy Loading (Q69, 70, 71)

**Q: What is Lazy vs Eager loading?**
- **Lazy**: Child entities are NOT fetched immediately. A Hibernate proxy is injected instead. The DB query is fired only when you explicitly call `parent.getChild()`. (Default for `@OneToMany` and `@ManyToMany`).
- **Eager**: Child entities are fetched immediately via a JOIN or supplementary SELECT. (Default for `@OneToOne` and `@ManyToOne`).

**Q: What is the N+1 problem and how to solve it?**
**The Problem**: If you fetch 10 `User` entities (1 query) and then iterate through them calling `user.getOrders()` (which is Lazy loaded), Hibernate will fire 10 separate queries to fetch the orders. 
Total Queries: 1 (to get users) + N (10 queries to get orders) = `N+1`.

**The Solution:**
1. **JOIN FETCH (JPQL):** Write a custom `@Query` to fetch all data simultaneously into memory.
   ```java
   @Query("SELECT u FROM User u JOIN FETCH u.orders")
   List<User> findAllUsersWithOrders();
   ```
2. **EntityGraphs**: Declaratively explicitly define which lazy relationships to pull eagerly for a specific query without writing JPQL.
   ```java
   @EntityGraph(attributePaths = {"orders"}, type = EntityGraphType.FETCH)
   List<User> findAll();
   ```

---

### Entity Lifecycle & Save Semantics (Q77, Q78, 79)

**Q: Entity lifecycle states?**
1. **Transient**: New POJO (using `new MyEntity()`), no DB mapping yet.
2. **Managed/Persistent**: Exists in the Persistence Context. (EntityManager is tracking it).
3. **Detached**: Transaction ended / Context cleared. Disconnected from DB. Changes on this object don't affect DB.
4. **Removed**: Tagged for deletion.

**Q: What is dirty checking?**
When an entity is in the **Managed** state, Hibernate takes a snapshot of it. When the transaction commits, Hibernate compares the current state to the snapshot. If it detects a change (it's "dirty"), Hibernate automatically issues a SQL `UPDATE`. You DO NOT need to call `repository.save()`.

**Q: Difference between save() and saveAndFlush()?**
- `save()`: Puts the entity into the persistence context but delays the actual execution of the SQL `INSERT/UPDATE` until it's absolutely necessary (usually at transaction commit time). This batches queries efficiently.
- `saveAndFlush()`: Bypasses batching. Forces the `EntityManager` to generate and execute the SQL query IMMEDIATELY to the database. Useful when you need the database triggers or constraints to run instantly.
