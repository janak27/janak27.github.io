# Java Streams: Advanced Grouping, Aggregation and Parallel Processing

This level covers ten practical Java Streams interview problems: finding maximum values, sorting, nested grouping, percentages, transaction aggregation, word frequency, index pairs, parallel streams, and `findFirst()` versus `findAny()`.

## 1. Find the highest-paid employee without sorting

```java
private static void findhighenosort(List<Employee> list) {
    Optional<Employee> emp = list.stream()
            .max(Comparator.comparingDouble(Employee::getSalary)); // Find the maximum salary without sorting

    System.out.println(emp); // Prints the highest-paid employee wrapped in Optional
}
```

`max()` compares the elements and returns an `Optional<Employee>`. An empty list produces `Optional.empty()`, so handle the `Optional` instead of assuming an employee exists.

## 2. Find the top two employees by salary in each department

```java
private static void toptwo(List<Employee> list) {
    Map<String, List<Employee>> ans = list.stream()
            .collect(Collectors.groupingBy(
                    Employee::getDepartment, // Group employees by department
                    Collectors.collectingAndThen(
                            Collectors.toList(),
                            employees -> employees.stream()
                                    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed()) // Sort salaries descending
                                    .limit(2) // Keep the top two employees in each department
                                    .toList()
                    )
            ));

    System.out.println(ans);
}
```

`groupingBy()` creates department groups. `collectingAndThen()` applies a finishing transformation after collecting each group to a list. Sorting followed by `limit(2)` retains up to two employees per department.

## 3. Sort employees by department, then salary descending

```java
private static void sortempbydepbysal(List<Employee> list) {
    List<Employee> ans = list.stream()
            .sorted(Comparator.comparing(Employee::getDepartment) // Sort departments alphabetically
                    .thenComparing(Comparator.comparingDouble(Employee::getSalary).reversed())) // Salary descending within each department
            .toList();

    System.out.println(ans);
}
```

`thenComparing()` adds a secondary comparison. Department order is ascending, and salary order within each department is descending. Reverse only the salary comparator; reversing the combined comparator would reverse the department order too.

## 4. Group employees by department and age range

```java
private static void grpdepgrpage(List<Employee> list) {
    Map<String, Map<String, List<Employee>>> ans = list.stream()
            .collect(Collectors.groupingBy(
                    Employee::getDepartment, // Group employees by department first
                    Collectors.groupingBy(emp -> {
                        int age = emp.getAge();

                        if (age < 25) return "Under 25"; // Younger than 25
                        if (age <= 35) return "25-35"; // Ages 25 through 35
                        if (age <= 45) return "36-45"; // Ages 36 through 45
                        return "Above 45"; // Older than 45
                    })
            ));

    System.out.println(ans);
}
```

Nested `groupingBy()` creates a map keyed first by department and then by age range. The ranges are Under 25, 25–35, 36–45, and Above 45.

## 5. Calculate each department's percentage of the workforce

```java
private static void findper(List<Employee> list) {
    long total = list.size(); // Get the total number of employees

    Map<String, Double> ans = list.stream()
            .collect(Collectors.groupingBy(
                    Employee::getDepartment,
                    Collectors.counting() // Count employees in each department
            ))
            .entrySet().stream()
            .collect(Collectors.toMap(
                    Map.Entry::getKey,
                    entry -> entry.getValue() * 100.0 / total // Convert department count to percentage
            ));

    System.out.println(ans); // IT=50.0, HR=25.0, Finance=25.0 (order may vary)
}
```

Formula: `(employees in department / total employees) * 100`. In the sample list, IT has 4 of 8 employees (50%), HR has 2 (25%), and Finance has 2 (25%). This code assumes the list is non-empty; handle `total == 0` before dividing if empty input is possible.

## 6. Find the customer with the highest total transaction amount

```java
private static void topspender(List<Transaction> tran) {
    Map.Entry<Integer, Double> ans = tran.stream()
            .collect(Collectors.groupingBy(
                    Transaction::getCustomerId,
                    Collectors.summingDouble(Transaction::getAmount) // Calculate each customer's total
            ))
            .entrySet().stream()
            .max(Map.Entry.comparingByValue()) // Find the customer with the highest total
            .orElse(null);

    System.out.println(ans); // Output: 104=1100.0
}
```

Sample totals:

| Customer ID | Total amount |
|---:|---:|
| 101 | 600.0 |
| 102 | 850.0 |
| 103 | 400.0 |
| 104 | 1100.0 |

Customer `104` is the top spender. `groupingBy()` groups transactions by customer, `summingDouble()` totals each customer's amounts, and `max()` finds the highest total. If the list is empty, `orElse(null)` returns `null`; consider retaining the result as an `Optional` if you want to avoid nulls.

## 7. Find the five most frequent words, ignoring case and punctuation

```java
private static void topfive(List<String> sentences) {
    List<Map.Entry<String, Long>> ans = sentences.stream()
            .flatMap(sentence -> Arrays.stream(sentence.toLowerCase()
                    .replaceAll("[^a-z0-9\\s]", "") // Remove punctuation
                    .split("\\s+"))) // Split sentences into words
            .filter(word -> !word.isBlank()) // Ignore empty words
            .collect(Collectors.groupingBy(
                    Function.identity(), Collectors.counting() // Count occurrences of each word
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed()
                    .thenComparing(Map.Entry.comparingByKey())) // Sort by frequency, then alphabetically
            .limit(5) // Keep the five most frequent words
            .toList();

    System.out.println(ans);
}
```

`flatMap()` turns the sentences into one stream of words. `groupingBy(Function.identity(), counting())` counts each word. Sorting by frequency descending and then alphabetically makes ties deterministic.

The pattern `[^a-z0-9\\s]` is suitable for basic English letters and digits; it does not preserve all accented or non-English letters. Use a Unicode-aware pattern for multilingual text.

## 8. Find all pairs whose sum equals a target

```java
private static void pairs(List<Integer> numbers, int target) {
    List<List<Integer>> ans = IntStream.range(0, numbers.size())
            .boxed()
            .flatMap(i -> IntStream.range(i + 1, numbers.size())
                    .filter(j -> numbers.get(i) + numbers.get(j) == target) // Check different indices
                    .mapToObj(j -> List.of(numbers.get(i), numbers.get(j)))) // Create the matching pair
            .toList();

    System.out.println(ans); // For target 7: [[1, 6], [3, 4], [5, 2]]
}
```

For `[1, 2, 3, 4, 5, 6, 2]` and target `7`, the matching value pairs are `[[1, 6], [3, 4], [5, 2]]`. Starting the inner range at `i + 1` prevents pairing an element with itself and avoids checking the same two indices in reverse order.

This solution takes \(O(n^2)\) time. Repeated values at different indices can produce repeated value pairs. A hash-based approach can be more efficient for large inputs.

## 9. Sum a list using a parallel stream

```java
private static void calculateSum(List<Integer> numbers) {
    long ans = numbers.parallelStream()
            .mapToLong(Integer::longValue) // Convert Integer values to primitive long
            .sum(); // Calculate the sum using parallel processing

    System.out.println(ans); // Output: 23
}
```

For `[1, 2, 3, 4, 5, 6, 2]`, the sum is `23`.

Parallel streams may help when the dataset and per-element work are large enough to offset splitting and coordination overhead. They may be slower for small collections, blocking I/O, or operations with shared-resource contention. Avoid shared mutable state and benchmark with realistic data; parallelism is not automatically faster.

## 10. Demonstrate `findFirst()` versus `findAny()`

```java
List<Integer> numbers2 = List.of(10, 20, 30, 40, 50);

System.out.println(numbers2.stream()
        .filter(n -> n > 20)
        .findFirst()); // Optional[30]: First match in encounter order

System.out.println(numbers2.stream()
        .filter(n -> n > 20)
        .findAny()); // Often Optional[30] for a sequential stream

List<Integer> numbers3 = List.of(10, 20, 30, 40, 50);

System.out.println(numbers3.parallelStream()
        .filter(n -> n > 20)
        .findFirst()); // Optional[30]: Preserves encounter order

System.out.println(numbers3.parallelStream()
        .filter(n -> n > 20)
        .findAny()); // May return any matching element, such as 30, 40, or 50
```

| Method | Behavior |
|---|---|
| `findFirst()` | Returns the first matching element in encounter order when one exists. |
| `findAny()` | Returns any matching element; the specific match is not guaranteed. |
| Sequential stream | `findAny()` often returns the first encountered match, but that is not guaranteed by its contract. |
| Parallel stream | `findAny()` may return a later match; `findFirst()` still preserves encounter order. |

Both return `Optional<T>` because there may be no matching element. Use `findFirst()` when encounter order matters; use `findAny()` when any match is acceptable.

## Quick revision

| Problem | Key API / technique |
|---|---|
| Highest-paid employee | `max(Comparator.comparingDouble(...))` |
| Top two per department | `groupingBy()`, `collectingAndThen()`, `sorted()`, `limit()` |
| Department then salary sorting | `comparing()`, `thenComparing()`, `reversed()` |
| Department and age-range groups | Nested `groupingBy()` |
| Department percentages | `groupingBy()`, `counting()`, percentage formula |
| Highest-spending customer | `groupingBy()`, `summingDouble()`, `max()` |
| Top five words | `flatMap()`, `groupingBy()`, `counting()`, sorting |
| Pairs matching a target | `IntStream.range()`, `flatMap()` |
| Parallel sum | `parallelStream()`, `mapToLong()`, `sum()` |
| First versus any match | `findFirst()`, `findAny()` |

## Interview questions to practise

1. Why is `max()` more suitable than sorting when only the highest-paid employee is required?
2. What does `collectingAndThen()` do in the top-two-per-department solution?
3. Why should `.reversed()` apply to the salary comparator rather than the combined comparator?
4. What is the return type of nested `groupingBy()`?
5. How would you handle an empty list when calculating department percentages?
6. Why use `summingDouble()` for transaction aggregation?
7. What does `flatMap()` contribute to word-frequency analysis?
8. What is the time complexity of checking every index pair?
9. Why can a parallel stream be slower than a sequential stream?
10. What guarantee does `findFirst()` provide that `findAny()` does not?

---

*Part 6 of the Java Streams API interview practice series.*
