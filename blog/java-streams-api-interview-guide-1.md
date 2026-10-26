# Java Streams: Core Concepts and Essential Operations

**Level:** Beginner to Interview Preparation  
**Language:** Java 8+  
**Topics:** `filter()`, `map()`, `reduce()`, `sorted()`, `distinct()`, `Optional`, and `Collectors`

## Introduction

The Java Streams API provides a declarative way to process data from collections. Instead of manually writing loops to filter, transform, sort, or aggregate elements, Streams allow us to express what operation we want to perform.

For example, suppose we want to extract even numbers from a list.

### Traditional Approach

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evenNumbers = new ArrayList<>();

for (Integer number : numbers) {
    if (number % 2 == 0) {
        evenNumbers.add(number);
    }
}

System.out.println(evenNumbers); // [2, 4, 6]
```

### Streams Approach

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

List<Integer> evenNumbers = numbers.stream()             // Create a Stream from the source list
        .filter(n -> n % 2 == 0)                         // Keep only even numbers
        .collect(Collectors.toList());                   // Collect matching elements into a new List

System.out.println(evenNumbers);                         // [2, 4, 6]
```

The Streams approach focuses on the transformation pipeline rather than manually managing the loop and result list.

## 1. Understanding the Stream Pipeline

A Stream pipeline consists of three main parts.

| Component | Purpose | Examples |
|---|---|---|
| Source | Provides the data | `list.stream()` |
| Intermediate operation | Filters, transforms, or orders elements | `filter()`, `map()`, `distinct()`, `sorted()` |
| Terminal operation | Triggers processing and produces a result | `collect()`, `count()`, `reduce()`, `forEach()` |

Example:

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

List<Integer> result = numbers.stream()                  // Source: create a Stream
        .filter(n -> n % 2 == 0)                         // Keep even numbers
        .map(n -> n * n)                                 // Square each remaining number
        .collect(Collectors.toList());                   // Execute the pipeline and collect results

System.out.println(result);                              // [4, 16, 36]
```

### Intermediate Operations Are Lazy

Intermediate operations do not process elements until a terminal operation triggers the pipeline.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3);

long count = numbers.stream()
        .filter(n -> {                                   // This filter runs when the pipeline is consumed
            System.out.println("Checking: " + n);
            return n > 1;
        })
        .count();                                        // Terminal operation triggers processing
```

### A Stream Cannot Be Reused

Once a terminal operation consumes a Stream, that Stream cannot be reused.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3);

Stream<Integer> stream = numbers.stream();               // Create a Stream
long count = stream.count();                             // Consume the Stream

// stream.forEach(System.out::println);                  // Invalid: the Stream has already been consumed
```

Create a new Stream from the original collection whenever another pipeline is required.

## 2. `filter()` — Select Elements

Use `filter()` when you want to keep elements that satisfy a condition.

### Example: Find Even Numbers

```java
static void evenNumbers(List<Integer> numbers) {
    List<Integer> result = numbers.stream()              // Convert the list into a Stream
            .filter(n -> n % 2 == 0)                     // Keep numbers divisible by 2
            .collect(Collectors.toList());               // Collect matching numbers into a new List

    System.out.println(result);                           // Print the even numbers
}
```

**Input:**

```text
[1, 2, 3, 4, 5, 6]
```

**Output:**

```text
[2, 4, 6]
```

### Example: Find Strings Longer Than Five Characters

```java
static void stringsLongerThanFive(List<String> words) {
    List<String> result = words.stream()                 // Convert the list into a Stream
            .filter(s -> s.length() > 5)                 // Keep strings with more than 5 characters
            .collect(Collectors.toList());               // Collect matching strings into a new List

    System.out.println(result);                           // Print the matching strings
}
```

**Input:** `["Java", "Spring", "Kafka", "Microservices"]`

**Output:** `[Spring, Microservices]`

### Example: Count Matching Strings

If the requirement asks for the number of matching elements rather than the elements themselves, use `count()`.

```java
static void countLongWords(List<String> words) {
    long count = words.stream()                          // Convert the list into a Stream
            .filter(s -> s.length() > 5)                 // Keep strings longer than 5 characters
            .count();                                    // Count matching elements and return a long

    System.out.println(count);                           // Print the count
}
```

**Output:** `2`

## 3. `map()` — Transform Elements

Use `map()` when you want to transform each element into another value.

### Example: Square Every Number

```java
static void squareNumbers(List<Integer> numbers) {
    List<Integer> result = numbers.stream()              // Convert the list into a Stream
            .map(n -> n * n)                             // Transform each number into its square
            .collect(Collectors.toList());               // Collect transformed numbers into a new List

    System.out.println(result);                           // Print the squared numbers
}
```

**Input:** `[1, 2, 3, 4, 5]`

**Output:** `[1, 4, 9, 16, 25]`

### Example: Convert Strings to Uppercase

```java
static void convertToUppercase(List<String> words) {
    List<String> result = words.stream()                 // Convert the list into a Stream
            .map(String::toUpperCase)                    // Convert each string to uppercase using a method reference
            .collect(Collectors.toList());               // Collect transformed strings into a new List

    System.out.println(result);                           // Print the uppercase strings
}
```

**Input:** `["Java", "Spring", "Kafka"]`

**Output:** `["JAVA", "SPRING", "KAFKA"]`

The equivalent lambda expression is `.map(s -> s.toUpperCase())`.

### Example: Convert Strings into Their Lengths

```java
static void stringLengths(List<String> words) {
    List<Integer> result = words.stream()                // Convert the list into a Stream
            .map(String::length)                         // Transform each string into its character count
            .collect(Collectors.toList());               // Collect lengths into a new List<Integer>

    System.out.println(result);                           // Print the string lengths
}
```

**Input:** `["Java", "Spring", "Kafka"]`

**Output:** `[4, 6, 5]`

### `filter()` vs `map()`

| Requirement | Operation | Example |
|---|---|---|
| Keep even numbers | `filter()` | `.filter(n -> n % 2 == 0)` |
| Keep strings longer than five characters | `filter()` | `.filter(s -> s.length() > 5)` |
| Square every number | `map()` | `.map(n -> n * n)` |
| Convert strings to uppercase | `map()` | `.map(String::toUpperCase)` |
| Convert strings to lengths | `map()` | `.map(String::length)` |

**Easy rule to remember:**

- Want to select or remove elements? Use `filter()`.
- Want to transform elements? Use `map()`.

### Common Mistake: Returning a Boolean Instead of a String

```java
// Incorrect when the result must be a List<String>
List<String> result = words.stream()
        .map(s -> s.length() > 5)                         // Returns boolean, not String
        .collect(Collectors.toList());
```

The expression `s.length() > 5` returns a Boolean. Consequently, `map()` creates a `Stream<Boolean>`, which cannot be collected into a `List<String>`.

To retain strings longer than five characters, use:

```java
List<String> result = words.stream()
        .filter(s -> s.length() > 5)                      // Keep matching strings
        .collect(Collectors.toList());                    // Collect them into a List<String>
```

## 4. `reduce()` — Combine Elements into One Result

Use `reduce()` when you want to combine multiple elements into a single result, such as a sum or product.

### Example: Sum All Integers

```java
static void sumNumbers(List<Integer> numbers) {
    int sum = numbers.stream()                            // Convert the list into a Stream
            .reduce(0, (a, b) -> a + b);                 // Start at 0 and accumulate each number into the sum

    System.out.println(sum);                              // Print the total sum
}
```

**Input:** `[1, 2, 3, 4, 5]`

**Output:** `15`

Understand `reduce(0, (a, b) -> a + b)`:

- `0` is the identity value and initial accumulated result.
- `a` represents the accumulated result so far.
- `b` represents the current element.
- `(a, b) -> a + b` combines the accumulated result with the current element.

| Step | Accumulated value (`a`) | Current element (`b`) | New result |
|---:|---:|---:|---:|
| 1 | 0 | 1 | 1 |
| 2 | 1 | 2 | 3 |
| 3 | 3 | 3 | 6 |
| 4 | 6 | 4 | 10 |
| 5 | 10 | 5 | 15 |

### Example: Multiply All Integers

```java
int product = numbers.stream()                           // Convert the list into a Stream
        .reduce(1, (a, b) -> a * b);                    // Start at 1 and multiply each number into the result

System.out.println(product);                             // Print the product
```

For `[1, 2, 3, 4, 5]`, the result is `120`.

The identity value is `1`, because multiplying a number by one does not change it.

## 5. Find Maximum and Minimum Values

The `max()` and `min()` operations return an `Optional<T>` because the Stream might be empty.

```java
static void findMaxAndMin(List<Integer> numbers) {
    int max = numbers.stream()                            // Create a Stream to find the maximum
            .max(Integer::compareTo)                      // Find the largest value and return an Optional
            .orElseThrow();                               // Return the value or throw if the list is empty

    int min = numbers.stream()                            // Create a fresh Stream to find the minimum
            .min(Integer::compareTo)                      // Find the smallest value and return an Optional
            .orElseThrow();                               // Return the value or throw if the list is empty

    System.out.println("Maximum: " + max);                // Print the largest value
    System.out.println("Minimum: " + min);                // Print the smallest value
}
```

**Input:** `[10, 5, 30, 2, 18]`

**Output:**

```text
Maximum: 30
Minimum: 2
```

### Handling an Empty List

Calling `orElseThrow()` without an argument throws `NoSuchElementException` if the `Optional` is empty.

If an empty list is valid input, handle it explicitly:

```java
Optional<Integer> max = numbers.stream()                  // Create a Stream from the list
        .max(Integer::compareTo);                         // Find the maximum, if one exists

max.ifPresent(value -> System.out.println(value));        // Print the maximum only when a value exists
```

## 6. `distinct()` — Remove Duplicate Elements

The `distinct()` operation removes duplicate elements. For an ordered Stream, it preserves encounter order.

```java
static void removeDuplicates(List<Integer> numbers) {
    List<Integer> result = numbers.stream()               // Convert the list into an ordered Stream
            .distinct()                                   // Keep one occurrence of each distinct value
            .collect(Collectors.toList());                // Collect unique values into a new List

    System.out.println(result);                            // Print the list without duplicates
}
```

**Input:** `[1, 2, 3, 2, 4, 1, 5, 3]`

**Output:** `[1, 2, 3, 4, 5]`

### Important Notes

- `distinct()` removes duplicates; it does not sort the elements.
- For objects, duplicate detection depends on `equals()` and `hashCode()`.
- The source list remains unchanged in this example.

## 7. `sorted()` — Sort Elements

### Ascending Order

```java
List<Integer> ascending = numbers.stream()               // Convert the list into a Stream
        .sorted()                                        // Sort in natural order, smallest to largest
        .collect(Collectors.toList());                   // Collect sorted values into a new List
```

### Descending Order

```java
List<Integer> descending = numbers.stream()              // Create a fresh Stream for descending order
        .sorted(Comparator.reverseOrder())               // Sort from largest to smallest
        .collect(Collectors.toList());                   // Collect sorted values into a new List
```

**Input:** `[5, 2, 8, 1, 9, 3]`

| Order | Output |
|---|---|
| Ascending | `[1, 2, 3, 5, 8, 9]` |
| Descending | `[9, 8, 5, 3, 2, 1]` |

The original list is not modified. Both pipelines collect their results into new lists.

## 8. `Collectors.joining()` — Join Strings

Use `Collectors.joining()` to combine strings into one string.

```java
static void joinStrings(List<String> words) {
    String result = words.stream()                      // Convert the list into a Stream
            .collect(Collectors.joining(","));          // Join strings using a comma separator

    System.out.println(result);                          // Print the combined string
}
```

**Input:** `["Java", "Spring", "Kafka"]`

**Output:** `Java,Spring,Kafka`

### Different Joining Options

```java
String plain = words.stream()
        .collect(Collectors.joining());                  // Join strings without a delimiter

String spaced = words.stream()
        .collect(Collectors.joining(", "));              // Join strings using comma-space separators

String wrapped = words.stream()
        .collect(Collectors.joining(", ", "[", "]"));   // Specify delimiter, prefix, and suffix
```

For `["Java", "Spring", "Kafka"]`, the last example produces:

```text
[Java, Spring, Kafka]
```

## 9. Complete Revision Class

This class combines the core examples in one place. It uses `Arrays.asList()` and `Collectors.toList()` for compatibility with Java 8 and later.

Save the file as `LevelOne.java` inside the `Fundamentals` package.

```java
package Fundamentals;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class LevelOne {

    public static void main(String[] args) {
        ArrayList<Integer> numbers = new ArrayList<>(
                Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 0, 10, 9, 8)); // Integer practice data

        ArrayList<String> words = new ArrayList<>(
                Arrays.asList("Java", "Spring", "Kafka", "Microservices")); // String practice data

        evenNumbers(numbers);                   // Select values using filter()
        squareNumbers(numbers);                 // Transform values using map()
        convertToUppercase(words);              // Convert strings to uppercase
        stringsLongerThanFive(words);           // Filter strings by length
        countLongWords(words);                  // Count matching strings
        sumNumbers(numbers);                    // Aggregate values using reduce()
        findMaxAndMin(numbers);                 // Find maximum and minimum
        removeDuplicates(numbers);              // Remove duplicates while preserving order
        sortNumbers(numbers);                   // Sort in both directions
        joinStrings(words);                     // Join strings into one value
        stringLengths(words);                   // Convert strings into their lengths
    }

    static void evenNumbers(List<Integer> numbers) {
        List<Integer> result = numbers.stream()             // Create a Stream from the list
                .filter(n -> n % 2 == 0)                    // Keep only even numbers
                .collect(Collectors.toList());              // Collect matching values into a new List

        System.out.println("Even numbers: " + result);      // Print even numbers
    }

    static void squareNumbers(List<Integer> numbers) {
        List<Integer> result = numbers.stream()             // Create a Stream from the list
                .map(n -> n * n)                            // Square each number
                .collect(Collectors.toList());              // Collect transformed values into a new List

        System.out.println("Squares: " + result);           // Print squared numbers
    }

    static void convertToUppercase(List<String> words) {
        List<String> result = words.stream()                // Create a Stream from the list
                .map(String::toUpperCase)                   // Convert each string to uppercase
                .collect(Collectors.toList());              // Collect transformed strings into a new List

        System.out.println("Uppercase: " + result);         // Print uppercase strings
    }

    static void stringsLongerThanFive(List<String> words) {
        List<String> result = words.stream()                // Create a Stream from the list
                .filter(s -> s.length() > 5)                // Keep strings longer than five characters
                .collect(Collectors.toList());              // Collect matching strings into a new List

        System.out.println("Long strings: " + result);      // Print matching strings
    }

    static void countLongWords(List<String> words) {
        long count = words.stream()                         // Create a Stream from the list
                .filter(s -> s.length() > 5)                // Keep strings longer than five characters
                .count();                                   // Count matching elements

        System.out.println("Long string count: " + count);  // Print the count
    }

    static void sumNumbers(List<Integer> numbers) {
        int sum = numbers.stream()                           // Create a Stream from the list
                .reduce(0, (a, b) -> a + b);               // Accumulate all numbers, starting from zero

        System.out.println("Sum: " + sum);                  // Print the total sum
    }

    static void findMaxAndMin(List<Integer> numbers) {
        Optional<Integer> max = numbers.stream()            // Create a Stream to find the maximum
                .max(Integer::compareTo);                   // Return the maximum as an Optional

        Optional<Integer> min = numbers.stream()            // Create a fresh Stream to find the minimum
                .min(Integer::compareTo);                   // Return the minimum as an Optional

        System.out.println("Maximum: " + max.orElse(null)); // Print the maximum, or null if empty
        System.out.println("Minimum: " + min.orElse(null)); // Print the minimum, or null if empty
    }

    static void removeDuplicates(List<Integer> numbers) {
        List<Integer> result = numbers.stream()             // Create an ordered Stream
                .distinct()                                 // Remove duplicates while preserving encounter order
                .collect(Collectors.toList());              // Collect unique values into a new List

        System.out.println("Unique values: " + result);     // Print unique values
    }

    static void sortNumbers(List<Integer> numbers) {
        List<Integer> ascending = numbers.stream()          // Create a Stream for ascending order
                .sorted()                                   // Sort from smallest to largest
                .collect(Collectors.toList());              // Collect sorted values into a new List

        List<Integer> descending = numbers.stream()         // Create a fresh Stream for descending order
                .sorted(Comparator.reverseOrder())          // Sort from largest to smallest
                .collect(Collectors.toList());              // Collect sorted values into a new List

        System.out.println("Ascending: " + ascending);      // Print ascending order
        System.out.println("Descending: " + descending);    // Print descending order
    }

    static void joinStrings(List<String> words) {
        String result = words.stream()                      // Create a Stream from the list
                .collect(Collectors.joining(", "));        // Join strings with comma-space separators

        System.out.println("Joined: " + result);            // Print the combined string
    }

    static void stringLengths(List<String> words) {
        List<Integer> result = words.stream()               // Create a Stream from the list
                .map(String::length)                        // Convert each string into its length
                .collect(Collectors.toList());              // Collect lengths into a new List<Integer>

        System.out.println("Lengths: " + result);            // Print the list of lengths
    }
}
```

## 10. Quick Reference Table

| Operation | Type | Result | Typical use |
|---|---|---|---|
| `filter()` | Intermediate | Stream of matching elements | Select values by condition |
| `map()` | Intermediate | Stream of transformed elements | Convert or extract values |
| `distinct()` | Intermediate | Stream of unique elements | Remove duplicates |
| `sorted()` | Intermediate | Sorted Stream | Order values |
| `limit(n)` | Intermediate | Stream of at most `n` elements | Take the first few values |
| `skip(n)` | Intermediate | Stream without the first `n` elements | Skip leading values |
| `count()` | Terminal | `long` | Count elements |
| `reduce()` | Terminal | Accumulated value | Sum or combine values |
| `min()` / `max()` | Terminal | `Optional<T>` | Find extremes |
| `collect()` | Terminal | Collected result | Build a list, set, map, or string |
| `forEach()` | Terminal | `void` | Perform an action for each element |
| `anyMatch()` | Terminal | `boolean` | Check whether any element matches |
| `allMatch()` | Terminal | `boolean` | Check whether every element matches |
| `noneMatch()` | Terminal | `boolean` | Check whether no element matches |

## 11. Common Interview Mistakes

1. **Using `map()` instead of `filter()`:** `map()` transforms elements, whereas `filter()` selects elements.
2. **Forgetting the terminal operation:** Intermediate operations are lazy, so a terminal operation is needed to execute the pipeline.
3. **Reusing a Stream:** Create a new Stream for every separate pipeline.
4. **Ignoring empty input:** `max()` and `min()` return empty `Optional` values when no elements exist.
5. **Confusing sorting with deduplication:** `sorted()` orders elements; `distinct()` removes duplicates.
6. **Assuming `count()` returns an `int`:** It returns a `long`.
7. **Assuming the original collection changes:** The operations shown here do not modify the source list.
8. **Assuming `Collectors.toList()` guarantees mutability:** The API does not guarantee the concrete type or mutability of the returned list.

## 12. Practice Questions

Try to solve these questions without looking at the examples above.

### Beginner

1. Find all odd numbers in a list.
2. Find all numbers greater than 10.
3. Convert every string to lowercase.
4. Find the length of each string.
5. Remove duplicate strings while preserving encounter order.
6. Sort integers in descending order.
7. Calculate the sum of integers using `reduce()`.
8. Find the maximum and minimum values.
9. Join words using `" - "` as the delimiter.
10. Count strings with more than five characters.

### Intermediate

11. Find the squares of even numbers only.
12. Find the sum of all even numbers.
13. Find the second-highest distinct number.
14. Check whether any number is negative using `anyMatch()`.
15. Check whether all numbers are positive using `allMatch()`.
16. Find the first string longer than five characters using `findFirst()`.
17. Skip the first two elements and return the next three.
18. Find duplicate values in a list.
19. Group strings by their length using `Collectors.groupingBy()`.
20. Count the frequency of each word using `Collectors.groupingBy()` and `Collectors.counting()`.

## Conclusion

The most useful starting pattern is:

```java
source.stream()                                           // Create a Stream from the source
        .filter(condition)                                // Optionally select elements
        .map(transformation)                              // Optionally transform elements
        .collect(Collectors.toList());                    // Execute and collect the result
```

Remember the mental model:

- **`filter()` selects** elements.
- **`map()` transforms** elements.
- **`reduce()` combines** elements.
- **`sorted()` orders** elements.
- **`distinct()` removes** duplicates.
- **Terminal operations trigger** the pipeline.

Once these fundamentals become familiar, practise combining operations in a single pipeline and move on to `Optional`, `flatMap()`, grouping, partitioning, custom collectors, and parallel Streams.