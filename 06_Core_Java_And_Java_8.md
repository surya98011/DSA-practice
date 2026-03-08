# Part 6: Core Java & Java 8 Internals

Interviews for L2 (7+ YOE) roles focus intensely on the *internals* of Core Java. They will not ask "how to use a HashMap", but rather "exactly how it resolves collisions dynamically."

---

### Collections Framework Internals

**Q: Explain the equals() vs hashCode() contract.**
The contract states:
1. If two objects are equal according to `equals()`, they MUST return the same `hashCode`.
2. If two objects have the same `hashCode`, they are NOT necessarily equal (Collisions can happen).
**Impact**: If you override `equals()` without overriding `hashCode()`, Collections like `HashSet` and `HashMap` will utterly fail. Two logically identical objects will hash to completely different buckets, meaning `map.get(key)` will return `null` instead of the required element.

**Q: Difference between HashMap and Hashtable?**
- `Hashtable` is deprecated legacy code (Java 1.0). Every single method in `Hashtable` is `synchronized`, making it completely thread-safe but horrendously slow. It does NOT allow `null` keys or values.
- `HashMap` is not thread-safe, operates at blazing speed, allows one `null` key and multiple `null` values. For thread safety, always use `ConcurrentHashMap`, which only locks specific segments/buckets, not the whole map.

**Q: Internal working of HashMap including hashing, buckets, collisions, and resizing?**
1. **Data Structure**: `HashMap` is essentially an array of Nodes/Buckets (default size 16).
2. **Put Process**: `map.put("a", 1)` evaluates `hash("a")`. It bitwise-ANDs the hash with `(size - 1)` to find the precise array index bucket.
3. **Collisions**: If two keys resolve to the exact same bucket, they are tied to each other via a Linked List.
4. **Java 8 Optimization**: If the Linked List in a single bucket reaches a threshold of `8` nodes (TREEIFY_THRESHOLD), the Linked List automatically transforms into a gracefully balanced **Red-Black Tree**. This guarantees `O(log n)` worst-case search times instead of devastating `O(n)` linear scans.
5. **Resizing**: When the map exceeds its load factor (default 75% full), it doubles the array size (32, 64) and re-hashes every existing entry, which is a very expensive CPU operation.

---

### Core Principles and Tricky Logic Questions

**Q: What is Method Hiding?**
If a subclass defines a `static` method with the exact same signature as a `static` method in the superclass, the subclass method *hides* the superclass method, it does NOT *override* it. Polymorphism/Dynamic Dispatch doesn't apply to statics; the method called is decided entirely by the class reference type at compile-time.

**Q: Can you override a static method in an interface?**
No. Default methods can be overridden, but static methods in interfaces cannot be inherited by implementing classes, they are hidden directly on the interface themselves.

**Q: Output of System.out.println(Double.MIN_VALUE > 0.0d);**
`true`. `Double.MIN_VALUE` is NOT a negative number. It is the smallest possible *positive* fraction greater than zero (`4.9e-324`). The real negative minimum is `-Double.MAX_VALUE`.

**Q: If you add a null value to an empty Set, what will be the size?**
`1`. A `HashSet` allows exactly one null element. It maps immediately to Bucket Index 0.

**Q: What is Comparable vs Comparator?**
- `Comparable` is in `java.lang`. You implement it on the class itself (`public class User implements Comparable<User>`), defining its natural sorting order via `compareTo(T)`.
- `Comparator` is in `java.util`. It is a separate interface allowing you to define multiple external sorting strategies (e.g., Sort Users by Age vs by Name) via `compare(T1, T2)`.

---

### Java 8 Deep Dive

**Q: What are the new features introduced in Java 8?**
1. Lambda Expressions.
2. Stream API (for functional collection bulk processing).
3. Functional Interfaces (`@FunctionalInterface`).
4. Default/Static methods in Interfaces.
5. Optional (Null-pointer prevention).
6. New Date/Time API (`java.time`).

**Q: What is a Functional Interface? Can we implement multiple interfaces having one abstract method?**
An interface holding exactly ONE abstract method. (It can contain any number of default or static methods). Useful as lambda targets.
Yes, a class can implement multiple functional interfaces. If two interfaces declare the exact same default method, the compiler will force the implementing class to manually override the method to resolve the absolute ambiguity.

**Q: What is a Method Reference?**
Shorthand for a lambda expression when the lambda literally does nothing but call an existing method.
`(user) -> System.out.println(user)` becomes `System.out::println`.

**Q: Explain Intermediate vs Terminal Operations in Streams.**
- **Intermediate**: `filter()`, `map()`, `sorted()`. They return *another pipeline Stream*. They are **lazy** and will never actually execute until a terminal operation activates them.
- **Terminal**: `collect()`, `forEach()`, `reduce()`, `count()`. They trigger the stream evaluation and consume the stream totally to return a non-stream result.

---

### Concurrency and JVM

**Q: Garbage Collector: What is it and how does it work?**
1. GC tracks which objects are still actively referenced by the application ("Live Roots", e.g., active thread stacks).
2. It reclaims Heap memory used by objects with zero active references.
3. Memory is split into **Young Generation** (Eden space, survived survivor spaces) and **Tenured/Old Generation**.
4. Modern GC algorithms: **G1GC** (Default in Java 9+, minimizes pause times asynchronously indexing regions), and **ZGC** (Java 15+ sub-millisecond terrabyte scalers).
