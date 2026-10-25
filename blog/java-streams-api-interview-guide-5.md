# Java Streams API: Flattening, Maps, Frequency Analysis and Custom Collectors - 5

This post covers ten practical Java Streams problems from
`Level05.java`, including nested collections, duplicate map keys,
frequency counting, map merging, consecutive sequences, and custom
aggregation.

> **Prerequisite:** The employee examples use the `Employee` class from
> the earlier practice set, with methods such as `getId()` and
> `getSalary()`.

## 1. Flatten a nested list using `flatMap()`

**Problem:** Convert a `List<List<Integer>>` into one `List<Integer>`.

``` java
private static void flatlist(List<List<Integer>> list1) {
    List<Integer> ans = list1.stream()
            .flatMap(List::stream) // Flatten each inner list into the outer stream
            .collect(Collectors.toList());

    System.out.println("Flattened list: " + ans);
}
```

Input: `[[1, 2, 3], [4, 5], [6, 7, 8]]`

Output: `[1, 2, 3, 4, 5, 6, 7, 8]`

`map()` would preserve the nested structure; `flatMap()` combines the
inner streams into one stream.

## 2. Flatten orders into a list of products

**Problem:** Each order contains products. Print every product name
across all orders.

``` java
private static void flatprodutsorders(List<Order> orders) {
    List<Product> products = orders.stream()
            .flatMap(order -> order.getProducts().stream()) // Flatten products from every order
            .collect(Collectors.toList());

    products.forEach(product -> System.out.println(product.getName()));
}
```

Output with the sample data:

``` text
Laptop
Mouse
Keyboard
Monitor
```

This is a common use of `flatMap()` when one object contains a
collection of child objects.

## 3. Convert employees into a map keyed by employee ID

**Problem:** Build a `Map<Integer, Employee>` using employee IDs as
keys.

``` java
private static void mapempid(List<Employee> list) {
    Map<Integer, Employee> map = list.stream()
            .collect(Collectors.toMap(
                    Employee::getId, // Use employee ID as the key
                    Function.identity(), // Store the complete Employee object
                    (existing, replacement) -> existing, // Keep the first value for duplicate IDs
                    LinkedHashMap::new // Preserve encounter order
            ));

    System.out.println("Employees by ID: " + map);
}
```

`Function.identity()` returns the input object unchanged.
`LinkedHashMap` preserves encounter order. The merge function prevents
duplicate IDs from causing `Collectors.toMap()` to throw
`IllegalStateException`.

## 4. Handle duplicate keys safely

The third argument to `Collectors.toMap()` is the merge function:

``` java
(existing, replacement) -> existing // Keep the existing value
```

To keep the later value, use `(existing, replacement) -> replacement`.
To combine values, provide a merge function that performs the required
combination.

**Source note:** Q3 and Q4 in the supplied file both call `mapempid()`.
The code handles duplicates, but these are not separate implementations.

## 5. Calculate total employee salaries using `mapToDouble()`

``` java
private static void allsalary(List<Employee> list) {
    double ans = list.stream()
            .mapToDouble(Employee::getSalary) // Convert to a primitive DoubleStream
            .sum();

    System.out.println("Total salary: " + ans);
}
```

With the eight sample employees, the total is `575000.0`.

`mapToDouble()` creates a primitive `DoubleStream`, making numeric
operations such as `sum()` convenient without boxing every salary into
`Double`.

## 6. Find the three most frequently occurring elements

``` java
private static void top3freq(List<Integer> numbers) {
    List<Integer> ans = numbers.stream()
            .collect(Collectors.groupingBy(
                    Function.identity(), // Group equal numbers
                    LinkedHashMap::new, // Preserve first-appearance order in the frequency map
                    Collectors.counting() // Count each occurrence
            ))
            .entrySet()
            .stream()
            .sorted(Map.Entry.<Integer, Long>comparingByValue().reversed()
                    .thenComparing(Map.Entry.comparingByKey())) // Break ties by ascending number
            .limit(3)
            .map(Map.Entry::getKey) // Extract the numbers
            .collect(Collectors.toList());

    System.out.println("Top three frequent elements: " + ans);
}
```

Input: `[1, 2, 2, 3, 3, 3, 4, 4, 4, 4, 5]`

Output: `[4, 3, 2]`

The solution groups values, counts each group, sorts by descending
frequency, breaks ties by ascending number, and takes at most three
values.

## 7. Find a missing number from `1` to `N`

``` java
private static void findmissing(List<Integer> numbers, int n) {
    int expectedSum = n * (n + 1) / 2; // Sum of integers from 1 through N

    int actualSum = numbers.stream()
            .mapToInt(Integer::intValue) // Convert Integer objects to primitive ints
            .sum();

    int ans = expectedSum - actualSum; // Difference gives the missing number
    System.out.println("Missing number: " + ans);
}
```

Input: `[1, 2, 3, 4, 5, 7]`, with `N = 7`

Output: `6`

**Assumption:** The input contains distinct numbers from `1` to `N`,
with exactly one number missing. For large ranges, use `long` to avoid
integer overflow.

## 8. Merge two maps by summing matching values

``` java
private static void mergemap(Map<String, Integer> map1, Map<String, Integer> map2) {
    Map<String, Integer> ans = Stream.concat(
                    map1.entrySet().stream(),
                    map2.entrySet().stream()
            )
            .collect(Collectors.toMap(
                    Map.Entry::getKey, // Use each entry's key
                    Map.Entry::getValue, // Use each entry's value
                    Integer::sum, // Add values for matching keys
                    LinkedHashMap::new // Preserve insertion order
            ));

    System.out.println("Merged map: " + ans);
}
```

For `{"Java": 10, "Python": 20}` and
`{"Java": 15, "Python": 5, "SQL": 8}`, the merged values are
`Java = 25`, `Python = 25`, and `SQL = 8`.

`Stream.concat()` combines the entry streams. `Integer::sum` defines how
duplicate keys are merged.

## 9. Find the longest consecutive sequence

``` java
private static void longconse(List<Integer> numbers) {
    Set<Integer> lookup = new HashSet<>(numbers); // Efficient membership checks

    List<Integer> ans = lookup.stream()
            .filter(n -> !lookup.contains(n - 1)) // Start only if the previous number is absent
            .map(n -> {
                List<Integer> sequence = new ArrayList<>();
                int current = n;

                while (lookup.contains(current)) {
                    sequence.add(current++); // Add current number and advance
                }

                return sequence;
            })
            .max(Comparator.comparingInt(List::size)) // Choose the longest sequence
            .orElse(Collections.emptyList()); // Handle empty input

    System.out.println("Longest consecutive sequence: " + ans);
}
```

Input: `[1, 2, 3, 4, 5, 7]`

Output: `[1, 2, 3, 4, 5]`

The key optimization is to start building a sequence only when `n - 1`
is absent. This avoids rebuilding the same sequence from every number
inside it. If multiple sequences tie for length, the code does not
define a specific tie-breaking rule.

## 10. Implement custom aggregation with `collect()`

``` java
private static void sumUsingCustomCollect(List<Integer> numbers) {
    int ans = numbers.stream()
            .collect(
                    () -> new int[1], // Supplier: create a mutable sum container
                    (sum, n) -> sum[0] += n, // Accumulator: add each number
                    (sum1, sum2) -> sum1[0] += sum2[0] // Combiner: merge partial sums
            )[0]; // Read the final sum

    System.out.println("Sum using custom collect: " + ans);
}
```

For `[1, 2, 3, 4, 5, 7]`, the result is `22`.

The three arguments are the **supplier**, **accumulator**, and
**combiner**. For a simple sum, `mapToInt(Integer::intValue).sum()` is
clearer; this example is useful for understanding how `collect()` works.

## Quick revision

  -----------------------------------------------------------------------
  Problem                             Main API or concept
  ----------------------------------- -----------------------------------
  Flatten nested lists                `flatMap()`

  Flatten orders into products        `flatMap()` with nested objects

  Map employees by ID                 `Collectors.toMap()`

  Handle duplicate keys               `toMap()` merge function

  Sum salaries                        `mapToDouble()` and `sum()`

  Top three frequencies               `groupingBy()`, `counting()`,
                                      sorting, `limit()`

  Find missing number                 `mapToInt()` and arithmetic

  Merge maps                          `Stream.concat()` and
                                      `Integer::sum`

  Longest consecutive sequence        `HashSet`, `filter()`, `max()`

  Custom aggregation                  Three-argument `collect()`
  -----------------------------------------------------------------------

## Interview questions

1.  What is the difference between `map()` and `flatMap()`?
2.  Why does `Collectors.toMap()` need a merge function when keys can
    repeat?
3.  What does `Function.identity()` return?
4.  Why use `mapToDouble()` or `mapToInt()` for numeric operations?
5.  What type does `Collectors.counting()` return?
6.  How does `Stream.concat()` help merge two maps?
7.  Why does the longest-sequence solution check that `n - 1` is absent?
8.  What are the supplier, accumulator, and combiner in the
    three-argument `collect()` method?

The goal is not just to memorize the code. Be ready to explain the
collector, the assumptions, and how you would change the solution if the
requirements changed.
