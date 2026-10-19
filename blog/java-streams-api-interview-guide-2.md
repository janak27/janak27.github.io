# Java Streams API: 10 Practical Interview Problems on Lists, Sets, Filtering, and Ranking

**Level:** Beginner to Intermediate\
**Language:** Java\
**Topics:** Streams API, `distinct()`, `sorted()`, `skip()`, `limit()`,
`map()`, `filter()`, `Optional`, `findFirst()`, `allMatch()`,
`anyMatch()`, `noneMatch()`, `groupingBy()`, `HashSet`, `LinkedHashSet`,
list intersection, and set difference

------------------------------------------------------------------------

## Introduction

Java Streams make it possible to express common collection operations as
a sequence of readable steps. Instead of manually looping through a
list, maintaining temporary variables, and checking conditions, we can
describe what we want to do: remove duplicates, sort values, filter
elements, transform objects, or collect the result.

This guide walks through ten practical problems using integers, strings,
and an `Employee` class. These are useful exercises for Java interviews
because they combine individual Stream operations into realistic
solutions.

The examples focus on **what each operation does, why its position in
the pipeline matters, and which edge cases to consider**.

## Test data

The main integer list intentionally contains positive numbers, negative
numbers, zeroes, and duplicates:

``` java
List<Integer> list = Arrays.asList(
    -1, 0, 1, -2, -2, -3, -4, 0, 1, 2, 3, 4,
    5, 6, 7, 8, 9, 10, 0, 10, 9, 8
);

List<Integer> list1 = Arrays.asList(11, 12, 13, 10, 9, 8);

List<String> list2 = Arrays.asList("Java", "Spring", "Kafka", "Microservices");
```

The employee data includes two employees with the same salary. This
helps demonstrate why ranking distinct salaries is different from
ranking employee records:

``` java
List<Employee> employees = Arrays.asList(
    new Employee("Amit", 50000),
    new Employee("Priya", 80000),
    new Employee("Rahul", 70000),
    new Employee("Neha", 80000),
    new Employee("Vijay", 60000)
);
```

## 1. Find the second-highest distinct number

### Problem

Find the second-largest **unique** number in a list.

### Solution

``` java
Optional<Integer> answer = list.stream()
    .distinct()                              // Remove duplicates before ranking
    .sorted(Comparator.reverseOrder())       // Sort from largest to smallest
    .skip(1)                                 // Skip the largest distinct value
    .findFirst();                            // Retrieve the next value, if present

System.out.println(answer.orElse(null));
```

**Output:**

``` text
9
```

### How it works

1.  `stream()` creates a stream from the list.
2.  `distinct()` removes repeated values.
3.  `sorted(Comparator.reverseOrder())` sorts the unique values in
    descending order.
4.  `skip(1)` skips the first value, which is the maximum.
5.  `findFirst()` returns the next value as an `Optional<Integer>`.

The distinct descending values begin with `10, 9, 8, 7, ...`, so the
second-highest distinct number is `9`.

### Why does `distinct()` come before `skip(1)`?

Suppose the list is `[10, 10, 9, 8]`.

-   Without `distinct()`, descending order is `[10, 10, 9, 8]`; skipping
    one element returns `10`.
-   With `distinct()`, the order is `[10, 9, 8]`; skipping one returns
    `9`.

**Interview takeaway:** When the question says *second-highest distinct
value*, remove duplicates before applying ranking operations.

### Edge case

If the list contains fewer than two distinct values, `findFirst()`
returns an empty `Optional`. `orElse(null)` prints `null` in that case.
In production code, returning `Optional<Integer>` from a method is often
preferable to using `null`.

------------------------------------------------------------------------

## 2. Find the third-highest distinct employee salary

### Problem

Find the third-highest unique salary across a collection of employees.

### Solution

``` java
Optional<Integer> answer = employees.stream()
    .map(Employee::getSalary)                // Transform each Employee into its salary
    .distinct()                              // Count each salary only once
    .sorted(Comparator.reverseOrder())       // Rank salaries from highest to lowest
    .skip(2)                                 // Skip the two highest distinct salaries
    .findFirst();                            // Retrieve the third-highest salary

System.out.println(answer.orElse(null));
```

**Output:**

``` text
60000
```

### How it works

The employee salaries are `50000, 80000, 70000, 80000, 60000`.

After `map(Employee::getSalary)`:

``` text
[50000, 80000, 70000, 80000, 60000]
```

After `distinct()` and descending sort:

``` text
[80000, 70000, 60000, 50000]
```

After `skip(2)`, the remaining values are:

``` text
[60000, 50000]
```

`findFirst()` retrieves `60000`.

### Why use `map()`?

The stream initially contains `Employee` objects. The ranking is based
on salary, so `map(Employee::getSalary)` converts each employee into an
`Integer` salary before the distinct and sorting operations run.

### Important distinction

This solution returns the **third-highest distinct salary**, not the
third employee after sorting. Priya and Neha both earn `80000`, but that
salary occupies only one rank.

------------------------------------------------------------------------

## 3. Find the top three highest distinct numbers

### Problem

Return up to three unique numbers in descending order.

### Solution

``` java
List<Integer> answer = list.stream()
    .distinct()                              // Keep each number once
    .sorted(Comparator.reverseOrder())       // Sort highest to lowest
    .limit(3)                                // Keep at most three results
    .collect(Collectors.toList());           // Collect results into a list

System.out.println(answer);
```

**Output:**

``` text
[10, 9, 8]
```

### `skip()` versus `limit()`

  -----------------------------------------------------------------------
  Operation               Purpose                 Example
  ----------------------- ----------------------- -----------------------
  `skip(n)`               Ignore the first `n`    `skip(2)` removes the
                          elements                first two

  `limit(n)`              Keep at most the first  `limit(3)` keeps at
                          `n` elements            most three
  -----------------------------------------------------------------------

The order matters. Here, `distinct()` and descending sorting happen
before `limit(3)`, so the result contains the top three unique numbers.

If the stream has fewer than three distinct values, `limit(3)` simply
returns all available values. It does not throw an exception.

------------------------------------------------------------------------

## 4. Find numbers whose string representation starts with `"1"`

### Problem

Return integers whose decimal string representation begins with the
character `"1"`.

### Solution

``` java
List<Integer> answer = list.stream()
    .filter(n -> String.valueOf(n).startsWith("1")) // Check the first character as text
    .collect(Collectors.toList());

System.out.println(answer);
```

**Output:**

``` text
[1, 1, 10, 10]
```

### Why convert a number to a string?

The condition is about the first **digit**, not the numeric value.
`String.valueOf(n)` converts each integer into text, and
`startsWith("1")` checks whether that text begins with `"1"`.

For example:

    Number String representation   Matches?
  -------- ----------------------- ----------
       `1` `"1"`                   Yes
      `10` `"10"`                  Yes
     `123` `"123"`                 Yes
      `21` `"21"`                  No
     `-10` `"-10"`                 No

This approach preserves the original integer values in the resulting
list. It does **not** remove duplicates, so repeated matching values
remain repeated.

**Interview note:** If the requirement is about a number's mathematical
range rather than its textual digits, use numeric comparisons instead.
For example, "between 10 and 19" is a different condition from "starts
with digit 1."

------------------------------------------------------------------------

## 5. Separate positive numbers, negative numbers, and zeroes

### Problem

Classify the list values into three categories.

### Approach A: Use three filters

``` java
List<Integer> positive = list.stream()
    .filter(n -> n > 0)                     // Keep numbers greater than zero
    .collect(Collectors.toList());

List<Integer> negative = list.stream()
    .filter(n -> n < 0)                     // Keep numbers less than zero
    .collect(Collectors.toList());

List<Integer> zeros = list.stream()
    .filter(n -> n == 0)                    // Keep zero values
    .collect(Collectors.toList());
```

This is straightforward and produces three separate lists. Duplicates
are preserved in each list because `filter()` does not remove
duplicates.

### Approach B: Group values with `groupingBy()`

``` java
Map<String, List<Integer>> grouped = list.stream()
    .collect(Collectors.groupingBy(n ->
        n > 0 ? "Positive" : n < 0 ? "Negative" : "Zero"
    ));

System.out.println(grouped);
```

The classifier expression assigns each number to one of three keys:

-   `Positive` when `n > 0`
-   `Negative` when `n < 0`
-   `Zero` otherwise

The resulting map contains a list of values for each category. **The
default map's key iteration order is not guaranteed**, so do not depend
on the order in which its entries print.

### Which approach should you choose?

-   Use separate filters when you need three named lists and want
    simple, explicit code.
-   Use `groupingBy()` when you want a map organized by a classification
    key, especially if the number of categories may grow.

Because each input number can belong to exactly one of these categories,
the groups are mutually exclusive and collectively cover the integer
values in the list.

------------------------------------------------------------------------

## 6. Find distinct elements common to two lists

### Problem

Find unique values that appear in both lists, preserving the encounter
order from the first list.

### Solution

``` java
Set<Integer> lookup = new HashSet<>(list1);  // Build a membership lookup set

List<Integer> answer = list.stream()
    .filter(lookup::contains)                // Keep values that occur in list1
    .distinct()                              // Return each common value once
    .collect(Collectors.toList());

System.out.println(answer);
```

**Output:**

``` text
[10, 9, 8]
```

### Why use a `HashSet`?

The second list is converted to a `HashSet`, allowing membership checks
with average-case constant-time lookup, commonly described as **O(1)**.
This avoids repeatedly scanning the second list for every element in the
first list.

The stream starts from `list`, so the output follows the first list's
encounter order. `distinct()` removes repeated common values.

### Complexity intuition

Let `n` be the size of the first list and `m` be the size of the second
list.

-   Building the lookup set takes expected `O(m)` time.
-   Filtering the first list takes expected `O(n)` membership-check
    time.
-   `distinct()` also maintains state to identify repeated values.

This is generally more efficient for membership checks than repeatedly
calling `list1.contains(n)` for every element of the first list.

------------------------------------------------------------------------

## 7. Find values unique to each list

### Problem

Find the values present in the first list but not the second, and vice
versa. In set terminology, these are the two directional differences.

### Solution

``` java
Set<Integer> lookup1 = new HashSet<>(list1); // Values available in the second list
Set<Integer> lookup = new HashSet<>(list);   // Values available in the first list

List<Integer> onlyInFirst = list.stream()
    .filter(n -> !lookup1.contains(n))       // Exclude values found in list1
    .distinct()                              // Keep each remaining value once
    .collect(Collectors.toList());

List<Integer> onlyInSecond = list1.stream()
    .filter(n -> !lookup.contains(n))        // Exclude values found in list
    .distinct()                              // Keep each remaining value once
    .collect(Collectors.toList());

System.out.println("Only in first list: " + onlyInFirst);
System.out.println("Only in second list: " + onlyInSecond);
```

**Output:**

``` text
Only in first list: [-1, 0, 1, -2, -3, -4, 2, 3, 4, 5, 6, 7]
Only in second list: [11, 12, 13]
```

### Understand the two directions

-   `onlyInFirst` starts with the first list and removes values that
    appear in the second list.
-   `onlyInSecond` starts with the second list and removes values that
    appear in the first list.

The `distinct()` operation ensures that repeated values in the original
list appear only once in the difference result. Each output retains the
encounter order of the list from which its stream was created.

**Interview distinction:** These results are not the same as the
*symmetric difference* in a single collection. The symmetric difference
combines both outputs into one collection.

------------------------------------------------------------------------

## 8. Find the first even number

### Problem

Find the first even number in encounter order.

### Solution

``` java
Optional<Integer> answer = list1.stream()
    .filter(n -> n % 2 == 0)                // Keep even numbers
    .findFirst();                            // Return the first matching element

System.out.println(answer.orElse(null));
```

**Output:**

``` text
12
```

The second list is `[11, 12, 13, 10, 9, 8]`. The first value is odd, and
`12` is the first even value.

### Why does `findFirst()` return an `Optional`?

There may be no matching element. `Optional<Integer>` represents either
a value or an empty result without requiring the method to return a
value in every case.

Common ways to handle it:

``` java
answer.ifPresent(System.out::println);       // Print only when a value exists
Integer value = answer.orElse(-1);           // Use -1 as a chosen fallback
Integer nullableValue = answer.orElse(null); // Return null when empty
```

Choose a fallback that makes sense for the application. A sentinel such
as `-1` is only safe if it cannot be confused with a valid result.

For an ordered stream, `findFirst()` respects encounter order.
`findAny()` is allowed to return any matching element and can be useful
in parallel processing when the exact first element is not required.

------------------------------------------------------------------------

## 9. Use `allMatch()`, `anyMatch()`, and `noneMatch()`

### Problem

Check whether all values are even, whether any value is odd, and whether
no value is negative.

### Solution

``` java
boolean allEven = list.stream()
    .allMatch(n -> n % 2 == 0);             // True only if every value is even

boolean anyOdd = list.stream()
    .anyMatch(n -> n % 2 != 0);             // True if at least one value is odd

boolean noNegative = list.stream()
    .noneMatch(n -> n < 0);                 // True if no value is negative

System.out.println("All even: " + allEven);
System.out.println("Any odd: " + anyOdd);
System.out.println("No negative: " + noNegative);
```

**Output:**

``` text
All even: false
Any odd: true
No negative: false
```

### Comparison

  ------------------------------------------------------------------------
  Operation                Returns `true` when     Stops early when
  ------------------------ ----------------------- -----------------------
  `allMatch(predicate)`    Every element matches   It finds a non-matching
                                                   element

  `anyMatch(predicate)`    At least one element    It finds a matching
                           matches                 element

  `noneMatch(predicate)`   No element matches      It finds a matching
                                                   element
  ------------------------------------------------------------------------

These are short-circuiting terminal operations: they can stop evaluating
as soon as the result is determined.

### Empty-stream edge case

For an empty stream:

-   `allMatch(...)` returns `true`.
-   `noneMatch(...)` returns `true`.
-   `anyMatch(...)` returns `false`.

This is intentional: an empty stream has no counterexample to `allMatch`
and no matching element for `anyMatch`. Keep this behavior in mind when
a collection might be empty.

------------------------------------------------------------------------

## 10. Convert a list into a set while preserving order

### Problem

Remove duplicates but keep the order in which each distinct value first
appeared.

### Solution

``` java
Set<Integer> answer = list.stream()
    .collect(Collectors.toCollection(LinkedHashSet::new)); // Unique values in encounter order

System.out.println(answer);
```

**Output:**

``` text
[-1, 0, 1, -2, -3, -4, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

### Why `LinkedHashSet` instead of `HashSet`?

  Collection        Removes duplicates   Preserves insertion order
  ----------------- -------------------- ------------------------------------------
  `HashSet`         Yes                  No guarantee
  `LinkedHashSet`   Yes                  Yes
  `TreeSet`         Yes                  Sorts by natural order or its comparator

A `LinkedHashSet` keeps the first occurrence of each value and preserves
the order in which unique values are inserted.
`Collectors.toCollection(LinkedHashSet::new)` explicitly chooses that
collection type.

**Important:** `distinct().collect(Collectors.toList())` is also a good
choice when the required result type is a list. Use `LinkedHashSet` when
the result should be a set with uniqueness enforced by the collection
itself.

------------------------------------------------------------------------

## Quick reference: the Stream operations used

  -------------------------------------------------------------------------------------
  Operation                             Category                Purpose
  ------------------------------------- ----------------------- -----------------------
  `distinct()`                          Intermediate            Removes duplicate
                                                                elements from the
                                                                stream

  `sorted()`                            Intermediate            Sorts elements in
                                                                natural order

  `sorted(Comparator.reverseOrder())`   Intermediate            Sorts in descending
                                                                order for comparable
                                                                values

  `map()`                               Intermediate            Transforms each element
                                                                into another value

  `filter()`                            Intermediate            Keeps elements that
                                                                satisfy a predicate

  `skip(n)`                             Intermediate            Skips the first `n`
                                                                elements

  `limit(n)`                            Intermediate            Keeps at most the first
                                                                `n` elements

  `collect()`                           Terminal                Accumulates elements
                                                                into a collection or
                                                                other result

  `findFirst()`                         Terminal                Returns the first
                                                                element as an
                                                                `Optional`

  `allMatch()`                          Terminal                Checks whether all
                                                                elements satisfy a
                                                                predicate

  `anyMatch()`                          Terminal                Checks whether at least
                                                                one element satisfies a
                                                                predicate

  `noneMatch()`                         Terminal                Checks whether no
                                                                element satisfies a
                                                                predicate

  `Collectors.groupingBy()`             Collector               Groups elements by a
                                                                classifier

  `Collectors.toCollection()`           Collector               Collects into a chosen
                                                                collection
                                                                implementation
  -------------------------------------------------------------------------------------

Intermediate operations build a pipeline; terminal operations trigger
its execution. A stream should not be reused after a terminal operation
has consumed it.

## Common interview mistakes

1.  **Skipping before removing duplicates.** For second- or
    third-highest *distinct* values, apply `distinct()` before ranking.
2.  **Ranking objects when the question asks for a field.** Use
    `map(Employee::getSalary)` before ranking salaries.
3.  **Assuming `filter()` removes duplicates.** It does not; add
    `distinct()` when unique results are required.
4.  **Assuming every set preserves order.** `HashSet` does not guarantee
    encounter order; `LinkedHashSet` preserves insertion order.
5.  **Ignoring an empty `Optional`.** A list may not contain enough
    distinct values or may have no matching element.
6.  **Assuming `groupingBy()` produces an ordered map.** The default
    map's iteration order is not guaranteed.
7.  **Confusing first match with any match.** `findFirst()` and
    `findAny()` have different guarantees.
8.  **Forgetting empty-stream match behavior.** `allMatch()` and
    `noneMatch()` return `true` for an empty stream; `anyMatch()`
    returns `false`.
9.  **Using a list for repeated membership checks unnecessarily.** A
    `HashSet` is often a better lookup structure when checking
    membership repeatedly.
10. **Misreading "top three."** Decide whether the requirement means
    three records or three distinct values before choosing the pipeline.

## Interview practice questions

Try answering these without looking at the solutions:

1.  How would you find the second-lowest distinct number?
2.  How would you find the highest salary without using `sorted()`?
3.  What changes if `distinct()` is removed from the third-highest
    salary pipeline?
4.  How would you return the top three employees by salary rather than
    the top three salary values?
5.  How would you count the number of positive, negative, and zero
    values using `groupingBy()`?
6.  How would you find the intersection of two lists while preserving
    duplicates from the first list?
7.  How would you calculate the symmetric difference of two lists?
8.  What is the difference between `findFirst()` and `findAny()`?
9.  Why can `allMatch()` return `true` for an empty stream?
10. When would you choose `TreeSet` over `LinkedHashSet`?

## Complete runnable example

The following version combines all ten solutions into one Java file.
Save it as `LevelTwo.java` inside a `Fundamentals` package folder. It
uses standard Java Streams and works with Java 8 or later.

``` java
package Fundamentals;

import java.util.*;
import java.util.stream.Collectors;

class Employee {
    private final String name;
    private final Integer salary;

    public Employee(String name, Integer salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public Integer getSalary() {
        return salary;
    }

    @Override
    public String toString() {
        return name + " : " + salary; // Make Employee objects readable when printed
    }
}

public class LevelTwo {

    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(
            -1, 0, 1, -2, -2, -3, -4, 0, 1, 2, 3, 4,
            5, 6, 7, 8, 9, 10, 0, 10, 9, 8
        );

        List<Integer> list1 = Arrays.asList(11, 12, 13, 10, 9, 8);

        List<String> list2 = Arrays.asList("Java", "Spring", "Kafka", "Microservices");

        List<Employee> employees = Arrays.asList(
            new Employee("Amit", 50000),
            new Employee("Priya", 80000),
            new Employee("Rahul", 70000),
            new Employee("Neha", 80000),
            new Employee("Vijay", 60000)
        );

        secondHighest(list);                   // Q1: Second-highest distinct number
        thirdHighestSalary(employees);         // Q2: Third-highest distinct salary
        topThreeHighest(list);                 // Q3: Top three distinct numbers
        numbersWithOne(list);                  // Q4: Numbers whose text starts with "1"
        separate(list);                        // Q5: Classify positive, negative, and zero
        commonNumbers(list, list1);            // Q6: Unique common values
        noCommon(list, list1);                 // Q7: Directional list differences
        firstEven(list1);                      // Q8: First even value
        allThings(list);                       // Q9: Match operations
        settingList(list);                     // Q10: Unique values in encounter order
    }

    private static void secondHighest(List<Integer> list) {
        Optional<Integer> answer = list.stream()
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst();

        System.out.println("Second highest: " + answer.orElse(null));
    }

    private static void thirdHighestSalary(List<Employee> employees) {
        Optional<Integer> answer = employees.stream()
            .map(Employee::getSalary)
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(2)
            .findFirst();

        System.out.println("Third-highest salary: " + answer.orElse(null));
    }

    private static void topThreeHighest(List<Integer> list) {
        List<Integer> answer = list.stream()
            .distinct()
            .sorted(Comparator.reverseOrder())
            .limit(3)
            .collect(Collectors.toList());

        System.out.println("Top three: " + answer);
    }

    private static void numbersWithOne(List<Integer> list) {
        List<Integer> answer = list.stream()
            .filter(n -> String.valueOf(n).startsWith("1"))
            .collect(Collectors.toList());

        System.out.println("Numbers starting with 1: " + answer);
    }

    private static void separate(List<Integer> list) {
        List<Integer> positive = list.stream()
            .filter(n -> n > 0)
            .collect(Collectors.toList());

        List<Integer> negative = list.stream()
            .filter(n -> n < 0)
            .collect(Collectors.toList());

        List<Integer> zeros = list.stream()
            .filter(n -> n == 0)
            .collect(Collectors.toList());

        System.out.println("Positive: " + positive);
        System.out.println("Negative: " + negative);
        System.out.println("Zeros: " + zeros);

        Map<String, List<Integer>> grouped = list.stream()
            .collect(Collectors.groupingBy(n ->
                n > 0 ? "Positive" : n < 0 ? "Negative" : "Zero"
            ));

        System.out.println("Grouped: " + grouped); // Map key iteration order is not guaranteed
    }

    private static void commonNumbers(List<Integer> list, List<Integer> list1) {
        Set<Integer> lookup = new HashSet<>(list1);

        List<Integer> answer = list.stream()
            .filter(lookup::contains)
            .distinct()
            .collect(Collectors.toList());

        System.out.println("Common elements: " + answer);
    }

    private static void noCommon(List<Integer> list, List<Integer> list1) {
        Set<Integer> lookup1 = new HashSet<>(list1);
        Set<Integer> lookup = new HashSet<>(list);

        List<Integer> onlyInFirst = list.stream()
            .filter(n -> !lookup1.contains(n))
            .distinct()
            .collect(Collectors.toList());

        List<Integer> onlyInSecond = list1.stream()
            .filter(n -> !lookup.contains(n))
            .distinct()
            .collect(Collectors.toList());

        System.out.println("Only in first list: " + onlyInFirst);
        System.out.println("Only in second list: " + onlyInSecond);
    }

    private static void firstEven(List<Integer> list) {
        Optional<Integer> answer = list.stream()
            .filter(n -> n % 2 == 0)
            .findFirst();

        System.out.println("First even: " + answer.orElse(null));
    }

    private static void allThings(List<Integer> list) {
        boolean allEven = list.stream().allMatch(n -> n % 2 == 0);
        boolean anyOdd = list.stream().anyMatch(n -> n % 2 != 0);
        boolean noNegative = list.stream().noneMatch(n -> n < 0);

        System.out.println("All even: " + allEven);
        System.out.println("Any odd: " + anyOdd);
        System.out.println("No negative: " + noNegative);
    }

    private static void settingList(List<Integer> list) {
        Set<Integer> answer = list.stream()
            .collect(Collectors.toCollection(LinkedHashSet::new));

        System.out.println("Unique elements: " + answer);
    }
}
```

### Final takeaways

-   Use `distinct()` before ranking when the requirement says
    **distinct**.
-   Combine `map()` with sorting when ranking a property such as
    employee salary.
-   Use `filter()` for conditions, and add `distinct()` separately when
    unique output is required.
-   Use `HashSet` for efficient membership checks and `LinkedHashSet`
    when uniqueness and insertion order both matter.
-   Handle potentially missing results with `Optional`.
-   Know the semantics of `allMatch()`, `anyMatch()`, and `noneMatch()`,
    including their behavior on empty streams.

The key interview skill is not memorizing a long pipeline. It is
translating the requirement into the right sequence of operations and
explaining why that sequence produces the intended result.
