# Java Streams API: Employee Grouping and Salary Analysis - 4

**Level:** Intermediate\
**Language:** Java\
**Topics:** `Collectors.groupingBy()`, `counting()`,
`averagingDouble()`, `maxBy()`, `minBy()`, `collectingAndThen()`,
`summingDouble()`, `partitioningBy()`, `mapping()`, and `Map.Entry`

------------------------------------------------------------------------

## Introduction

This is the fourth part of the Java Streams API interview-practice
series. This guide moves from string operations to working with objects.
Using an `Employee` class, we will group employees by department,
calculate salary statistics, find the highest- and lowest-paid
employees, partition employees by salary, and transform employee records
into lists of names.

These problems are useful for practising downstream collectors: instead
of collecting every group into a list, we can calculate a count,
average, total, or selected employee for each group.

## Employee model and test data

The examples use this `Employee` class:

``` java
class Employee {
    private int id;
    private String name;
    private String department;
    private double salary;
    private int age;

    public Employee(int id, String name, String department, double salary, int age) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
        this.age = age;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public int getAge() { return age; }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", department='" + department + '\'' +
                ", salary=" + salary +
                ", age=" + age +
                '}';
    }
}
```

The test data contains employees from three departments, including two
IT employees with the same salary:

``` java
List<Employee> list = List.of(
    new Employee(1, "Amit", "IT", 75000, 28),
    new Employee(2, "Priya", "HR", 55000, 25),
    new Employee(3, "Rahul", "IT", 90000, 32),
    new Employee(4, "Sneha", "Finance", 65000, 29),
    new Employee(5, "Vikas", "HR", 60000, 35),
    new Employee(6, "Neha", "IT", 85000, 26),
    new Employee(7, "Karan", "Finance", 70000, 31),
    new Employee(8, "Pooja", "IT", 75000, 24)
);
```

`List.of()` is available from Java 9 onward and creates an unmodifiable
list. The employee objects themselves are not immutable in this example,
because their fields are not declared `final`.

## 1. Group employees by department

### Problem

Group employees so that each department maps to a list of its employees.

### Solution

``` java
private static void groupthem(List<Employee> list) {
    Map<String, List<Employee>> map = list.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));

    map.forEach((department, employeeList) ->
        System.out.println(department + " -> " + employeeList));
}
```

**Expected grouping:**

-   `IT` → Amit, Rahul, Neha, Pooja
-   `HR` → Priya, Vikas
-   `Finance` → Sneha, Karan

The full printed output includes each employee's fields because
`Employee.toString()` is overridden.

### How it works

-   `Employee::getDepartment` is the classifier function.
-   `groupingBy()` collects each employee under the value returned by
    that function.
-   The default downstream collector stores each department's employees
    in a list.

The map's iteration order is not guaranteed by the default
`groupingBy()` collector. If output order matters, specify a map
supplier such as `LinkedHashMap::new`.

## 2. Count employees in each department

### Problem

Return a `Map<String, Long>` containing the number of employees in each
department.

### Solution

``` java
private static void countbydept(List<Employee> list) {
    Map<String, Long> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.counting()
        ));

    map.forEach((department, count) ->
        System.out.println(department + " -> " + count));
}
```

**Expected values:**

``` text
IT -> 4
HR -> 2
Finance -> 2
```

### Key concept: downstream collectors

The first argument to `groupingBy()` identifies the group. The second
argument tells Java what to calculate for each group.

Here, `Collectors.counting()` counts the employees in each department
and returns a `Long`, which is why the map type is `Map<String, Long>`
rather than `Map<String, Integer>`.

## 3. Calculate the average salary of each department

### Problem

Calculate the average employee salary separately for each department.

### Solution

``` java
private static void getavg(List<Employee> list) {
    Map<String, Double> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.averagingDouble(Employee::getSalary)
        ));

    map.forEach((department, salary) ->
        System.out.println(department + " -> " + salary));
}
```

**Expected values:**

  Department     Average salary
  ------------ ----------------
  IT                    81250.0
  HR                    57500.0
  Finance               67500.0

### How it works

`averagingDouble(Employee::getSalary)` extracts each employee's salary
and calculates the average for the group. The result is a `Double` for
each department.

For IT, the calculation is:

\[ `\frac{75000 + 90000 + 85000 + 75000}{4}`{=tex} = 81250 \]

This pattern is useful for average order values by customer, average
marks by subject, or average transaction amounts by category.

## 4. Find the highest-paid employee in each department

### Problem

Return the employee with the maximum salary in each department.

### Solution

``` java
private static void highestpaid(List<Employee> list) {
    Map<String, Optional<Employee>> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary))
        ));

    map.forEach((department, employee) ->
        System.out.println("Highest paid in " + department + " -> "
            + employee.orElse(null)));
}
```

**Expected highest-paid employees:**

-   `IT` → Rahul, salary `90000.0`
-   `HR` → Vikas, salary `60000.0`
-   `Finance` → Karan, salary `70000.0`

### Why does the map contain `Optional<Employee>`?

`Collectors.maxBy()` returns an `Optional<Employee>`. The map therefore
has the type `Map<String, Optional<Employee>>`.

Each department created by `groupingBy()` has at least one employee in
this input, so each optional contains a result. The `orElse(null)` call
is used here for printing; in application code, consider keeping the
`Optional` or unwrapping it deliberately.

## 5. Find the lowest-paid employee in each department

### Problem

Return the employee with the minimum salary in each department.

### Solution

``` java
private static void lowestpaid(List<Employee> list) {
    Map<String, Optional<Employee>> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.minBy(Comparator.comparingDouble(Employee::getSalary))
        ));

    map.forEach((department, employee) ->
        System.out.println("Lowest paid in " + department + " -> "
            + employee.orElse(null)));
}
```

**Expected lowest-paid employees:**

-   `IT` → Amit or Pooja, both with salary `75000.0`
-   `HR` → Priya, salary `55000.0`
-   `Finance` → Sneha, salary `65000.0`

Amit and Pooja have equal salaries. `minBy()` returns one matching
minimum employee; if a particular tie-breaking rule is required, define
it in the comparator.

### `minBy()` versus `maxBy()`

The pipeline is identical to Question 4 apart from the terminal
collector:

-   `maxBy()` selects the maximum according to the comparator.
-   `minBy()` selects the minimum according to the comparator.

## 6. Find the second-highest-paid employee in each department

### Problem

Find the employee after the highest-paid employee when employees are
sorted by salary in descending order within each department.

### Solution

``` java
private static void secondhighest(List<Employee> list) {
    Map<String, Optional<Employee>> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.collectingAndThen(
                Collectors.toList(),
                employees -> employees.stream()
                    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
                    .skip(1)
                    .findFirst()
            )
        ));

    map.forEach((department, employee) ->
        System.out.println("Second highest in " + department + " -> "
            + employee.orElse(null)));
}
```

**Expected result for the supplied data:**

-   `IT` → Neha, salary `85000.0`
-   `HR` → Priya, salary `55000.0`
-   `Finance` → Sneha, salary `65000.0`

### How it works

1.  `groupingBy()` creates a group for each department.
2.  `collectingAndThen()` applies a second operation after collecting
    each group's employees into a list.
3.  `sorted(...reversed())` orders employees from highest salary to
    lowest.
4.  `skip(1)` skips the first employee.
5.  `findFirst()` returns the next employee, if present.

### Important interview distinction: second employee or second distinct salary?

This solution finds the second employee in the sorted list. It does
**not** find the second-highest distinct salary. If two employees share
the highest salary, `skip(1)` can return another employee with that same
salary.

To solve the second-highest **distinct salary** problem, clarify the
requirement and deduplicate salary values before selecting the second
value---or filter employees by the second distinct salary, depending on
the expected return type.

## 7. Calculate total salary expenditure for each department

### Problem

Add the salaries of all employees in each department.

### Solution

``` java
private static void totalsalary(List<Employee> list) {
    Map<String, Double> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.summingDouble(Employee::getSalary)
        ));

    map.forEach((department, salary) ->
        System.out.println(department + " -> " + salary));
}
```

**Expected values:**

  Department     Total salary
  ------------ --------------
  IT                 325000.0
  HR                 115000.0
  Finance            135000.0

### How it works

`Collectors.summingDouble(Employee::getSalary)` adds the salary values
within each department and returns a `Double`.

This pattern is useful for total payroll, total sales per region, or
total transaction amounts per account type.

## 8. Partition employees by whether salary exceeds ₹60,000

### Problem

Split employees into two groups: those earning more than ₹60,000 and
those earning ₹60,000 or less.

### Solution

``` java
private static void partby60K(List<Employee> list) {
    Map<Boolean, List<Employee>> map = list.stream()
        .collect(Collectors.partitioningBy(
            employee -> employee.getSalary() > 60000
        ));

    System.out.println("Salary above ₹60,000: " + map.get(true));
    System.out.println("Salary ₹60,000 or below: " + map.get(false));
}
```

The predicate is `employee.getSalary() > 60000`, so an employee earning
exactly ₹60,000 belongs to the `false` group.

**Employees above ₹60,000:** Amit, Rahul, Sneha, Neha, Karan, Pooja.\
**Employees earning ₹60,000 or below:** Priya, Vikas.

### `partitioningBy()` versus `groupingBy()`

-   `partitioningBy()` divides the stream into two groups based on a
    boolean predicate.
-   `groupingBy()` groups elements using a classifier that can return
    different keys, such as department names.

Use partitioning when the requirement naturally has two outcomes, such
as eligible/ineligible or passed/failed.

## 9. Find the department with the highest average salary

### Problem

Calculate each department's average salary and return the department
with the largest average.

### Solution

``` java
private static void deptwithhighsalary(List<Employee> list) {
    Map.Entry<String, Double> result = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.averagingDouble(Employee::getSalary)
        ))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue())
        .orElseThrow();

    System.out.println("Department with highest average salary: "
        + result.getKey() + " -> " + result.getValue());
}
```

**Output:**

``` text
Department with highest average salary: IT -> 81250.0
```

### Pipeline breakdown

1.  Group employees by department and calculate each average.
2.  `entrySet()` exposes the department-average pairs.
3.  `stream()` allows the entries to be processed.
4.  `Map.Entry.comparingByValue()` compares entries by average salary.
5.  `max()` selects the entry with the largest average.
6.  `orElseThrow()` makes the empty-input behavior explicit by throwing
    if there are no departments.

For code that should handle empty input gracefully, return an
`Optional<Map.Entry<String, Double>>` instead of calling
`orElseThrow()`.

## 10. Get employee names by department

### Problem

Return a `Map<String, List<String>>`, where each department maps to the
names of its employees.

### Solution

``` java
private static void findnames(List<Employee> list) {
    Map<String, List<String>> map = list.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.mapping(
                Employee::getName,
                Collectors.toList()
            )
        ));

    map.forEach((department, names) ->
        System.out.println(department + " -> " + names));
}
```

**Expected grouping:**

-   `IT` → `[Amit, Rahul, Neha, Pooja]`
-   `HR` → `[Priya, Vikas]`
-   `Finance` → `[Sneha, Karan]`

### Why use `mapping()`?

Without `mapping()`, the default result of grouping is a list of
complete `Employee` objects.
`Collectors.mapping(Employee::getName, Collectors.toList())` transforms
each employee into a name before collecting the names into a list.

This is a common interview pattern for extracting fields while grouping:
names by department, product IDs by category, or transaction references
by account.

## Quick comparison

  -----------------------------------------------------------------------
  Requirement                         Main collector or operation
  ----------------------------------- -----------------------------------
  Group employees by department       `groupingBy()`

  Count employees per department      `counting()`

  Average salary by department        `averagingDouble()`

  Highest-paid employee per           `maxBy()`
  department                          

  Lowest-paid employee per department `minBy()`

  Second employee by descending       `collectingAndThen()`, `sorted()`,
  salary                              `skip()`, `findFirst()`

  Total salary by department          `summingDouble()`

  Split employees into two groups     `partitioningBy()`

  Department with highest average     `Map.Entry.comparingByValue()` +
                                      `max()`

  Names by department                 `mapping()` + `toList()`
  -----------------------------------------------------------------------

## Common mistakes to avoid

1.  **Using the wrong map value type:** `counting()` returns `Long`, and
    `averagingDouble()` and `summingDouble()` return `Double`.
2.  **Forgetting the downstream collector:** `groupingBy()` can produce
    a list by default, or a different result when given a downstream
    collector.
3.  **Confusing `partitioningBy()` with `groupingBy()`:** partitioning
    always uses a boolean predicate.
4.  **Assuming `skip(1)` means second distinct salary:** it skips one
    employee record, not one unique salary value.
5.  **Ignoring ties:** multiple employees can share the same salary. Add
    a tie-breaker if a particular employee must be selected.
6.  **Assuming map iteration order:** the default `groupingBy()` does
    not promise department order.
7.  **Not handling empty streams:** `max()` and `findFirst()` return an
    `Optional`; decide how an absent result should be handled.
8.  **Using `List.of()` on Java 8:** `List.of()` requires Java 9 or
    later. For Java 8, use `Arrays.asList()` or another list
    implementation.

## Interview practice questions

Try solving these without looking at the solutions above:

1.  Find the second-highest **distinct salary** in each department.
2.  Find the top three employees by salary in each department.
3.  Count employees in each department whose age is above 30.
4.  Calculate the minimum, maximum, and average salary for every
    department.
5.  Group employees by department and sort each group's names
    alphabetically.
6.  Find the department with the lowest average salary.
7.  Partition employees by age greater than or equal to 30.
8.  Return each department's employee names as a comma-separated string.
9.  Find departments whose total salary expenditure exceeds ₹100,000.
10. Explain the difference between `groupingBy()`, `partitioningBy()`,
    and `mapping()`.

## Complete runnable example

Place the `Employee` class and the `Level04` class in `Level04.java`
inside the `Fundamentals` package. The supplied class contains all ten
methods. Uncomment one method call at a time in `main()` to practise the
corresponding operation.

Required imports:

``` java
import java.util.*;
import java.util.stream.Collectors;
```

The examples use Java 9's `List.of()`. To run them on Java 8, replace it
with `Arrays.asList(...)`.

## Conclusion

These examples show how Java Streams can turn employee-processing tasks
into compact, readable pipelines. The key is to understand both the
grouping classifier and the downstream collector: the classifier chooses
the group, while the collector calculates what you need from each group.

For interviews, practise explaining why you chose `counting()`,
`averagingDouble()`, `maxBy()`, `summingDouble()`, or `mapping()`. Also
be precise about ranking requirements---especially the difference
between the second employee by salary and the second-highest distinct
salary.
