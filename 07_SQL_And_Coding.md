# Part 7: SQL Queries & Hands-on Coding

Practical implementation queries and the explicit Array coding challenge requested in the prompt.

---

### SQL Database Queries 

**Q: You have a user table with 50 records. Write a query to fetch 20 records starting from 5th row (first & last name only).**
*(Often asked to test your knowledge of dynamic pagination on the database side).*

```sql
-- Standard ANSI SQL / PostgreSQL / MySQL approach
SELECT first_name, last_name
FROM users
ORDER BY id ASC
LIMIT 20 OFFSET 4; 
-- OFFSET is 0-indexed, so skipping 4 rows means starting at row 5.
```

**Q: Write an INNER JOIN query between two tables.**
If there are two tables mapping a One-to-Many generic relationship (e.g., 1 User has many Orders):

```sql
SELECT u.first_name, u.last_name, o.order_id, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- INNER JOIN specifically guarantees that ONLY users who have actual orders will appear 
-- in the result set. If a user has zero orders, they are completely excluded. To include them, use LEFT JOIN.
```

### Core Coding Challenge (Arrays and Streams)

**Q: Write a program to find numbers ending with 6 in the given two-digit integer array. `int[] arr = {11,22,33,44,63,66};`**

As a 7 YOE candidate, you should provide both the traditional approach (for pure performance without object boxing) AND the Java 8 Streams approach (to show modern familiarity).

#### Approach 1: Java 8 Streams API (Cleaner & Declarative)

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class ArrayFilter {
    public static void main(String[] args) {
        int[] arr = {11, 22, 33, 44, 63, 66};

        // Convert primitives into IntStream, filter them by mathematical modulo logic, and collect them via boxed Integers.
        List<Integer> result = Arrays.stream(arr)
                                     .filter(num -> num % 10 == 6)
                                     .boxed()
                                     .collect(Collectors.toList());

        System.out.println("Numbers ending with 6: " + result); // Output: [66]
    }
}
```

#### Approach 2: Traditional Pre-Java 8 Approach (Higher Performance)

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayFilterStandard {
    public static void main(String[] args) {
        int[] arr = {11, 22, 33, 44, 63, 66};
        List<Integer> result = new ArrayList<>();

        for (int num : arr) {
            // Check mathematically if the last digit is 6 using the modulus operator.
            if (num % 10 == 6) {
                result.add(num);
            }
        }

        System.out.println("Numbers ending with 6: " + result); // Output: [66]
    }
}
```

*(Note for the interviewer: Modulus operator `% 10` is mathematically the fastest method to analyze trailing digits, far superior to parsing integers into String equivalents and checking `.endsWith("6")`, which wastes memory on huge O(n) String pool allocations).*
