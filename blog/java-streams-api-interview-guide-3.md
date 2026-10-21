# Java Streams API: Practical Interview Problems - 3

**Level:** Beginner to Intermediate\
**Language:** Java\
**Topics:** Streams API, character frequency, duplicate detection,
anagrams, `Optional`, `Comparator`, and `Collectors.groupingBy()`.

------------------------------------------------------------------------

## Introduction

This guide continues the Java Streams interview series with 11 practical
string-processing problems. Each example demonstrates a reusable stream
pattern and explains what to discuss in an interview.

## Test data

``` java
String s = "JanakBabanAvhad";
String s1 = "JanakAvhad";
String s2 = "AvhadJanak";
String sentence = " Bharat is iss my  maiyy ccouct country";
```

The sentence contains leading and repeated spaces. For word processing,
`trim()` and `split("\\s+")` remove the effect of that extra whitespace.

## Reusable character-frequency pattern

``` java
Map<Character, Long> freq = s.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(
        Function.identity(), LinkedHashMap::new, Collectors.counting()
    ));
```

-   `chars()` produces an `IntStream`.
-   `mapToObj()` converts each integer into a `Character`.
-   `groupingBy()` groups equal characters.
-   `LinkedHashMap::new` preserves first-appearance order.
-   `counting()` returns a `Long` count.

This pattern is case-sensitive: `A` and `a` are different characters.

## 1. Count the frequency of each character

``` java
private static void countfreq(String s) {
    Map<Character, Long> freq = s.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));
    System.out.println("Character frequency: " + freq);
}
```

**Expected output:**

``` text
{J=1, a=5, n=2, k=1, B=1, b=1, A=1, v=1, h=1, d=1}
```

Use `Function.identity()` when the character itself should be the key.
`LinkedHashMap` retains key insertion order; it does not sort the keys.

## 2. Find the first non-repeating character

``` java
private static void nonrepeating(String s) {
    Map<Character, Long> freq = s.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    Optional<Character> answer = s.chars()
        .mapToObj(c -> (char) c)
        .filter(c -> freq.get(c) == 1)
        .findFirst();

    System.out.println(answer.orElse(null));
}
```

**Output:** `J`

The frequency map identifies unique characters; streaming the original
string again and calling `findFirst()` selects the earliest one.
`Optional` handles the case where no character matches.

## 3. Find the first repeating character

``` java
private static void firstrepeating(String s) {
    Map<Character, Long> freq = s.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    Optional<Character> answer = s.chars()
        .mapToObj(c -> (char) c)
        .filter(c -> freq.get(c) > 1)
        .findFirst();

    System.out.println(answer.orElse(null));
}
```

**Output:** `a`

This finds the first character from left to right whose total frequency
is greater than one. For `abca`, it returns `a`; it does not
specifically find the character at the position of its second
occurrence.

## 4. Find all duplicate characters

``` java
private static void allrepeating(String s) {
    Map<Character, Long> freq = s.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    List<Character> answer = s.chars()
        .mapToObj(c -> (char) c)
        .filter(c -> freq.get(c) > 1)
        .distinct()
        .toList();

    System.out.println(answer);
}
```

**Output:** `[a, n]`

`filter()` keeps every occurrence of a duplicated character.
`distinct()` ensures each duplicated character appears only once.
`Stream.toList()` returns an unmodifiable list on Java 16+.

## 5. Find the character(s) with the highest frequency

``` java
private static void mostrepeating(String s) {
    Map<Character, Long> freq = s.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    long max = freq.values().stream()
        .mapToLong(Long::longValue)
        .max()
        .orElse(0);

    List<Character> answer = freq.entrySet().stream()
        .filter(e -> e.getValue() == max)
        .map(Map.Entry::getKey)
        .toList();

    System.out.println(answer + " with frequency " + max);
}
```

**Output:** `[a] with frequency 5`

`mapToLong()` unboxes the counts into primitive `long` values. `max()`
finds the largest count; filtering the entries retrieves every character
tied at that count. `orElse(0)` handles an empty string.

## 6. Find the most frequently occurring character(s)

The `mostfreq()` method in the original practice class performs the same
calculation as Question 5. The method names differ, but the requirements
and algorithms are identical. You only need one implementation unless
you are deliberately practising naming variations.

``` java
private static void mostfreq(String s) {
    Map<Character, Long> freq = s.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    long max = freq.values().stream()
        .mapToLong(Long::longValue)
        .max()
        .orElse(0);

    List<Character> answer = freq.entrySet().stream()
        .filter(e -> e.getValue() == max)
        .map(Map.Entry::getKey)
        .toList();

    System.out.println(answer + " with frequency " + max);
}
```

**Output:** `[a] with frequency 5`

Interview tip: clarify the requirement before implementing two
differently worded questions that ask for the same result.

## 7. Check whether two strings are anagrams

``` java
private static void checkanagrams(String s1, String s2) {
    String str1 = s1.replaceAll("\\s+", "").toLowerCase();
    String str2 = s2.replaceAll("\\s+", "").toLowerCase();

    Map<Character, Long> freq1 = str1.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    Map<Character, Long> freq2 = str2.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(), LinkedHashMap::new, Collectors.counting()
        ));

    System.out.println(freq1.equals(freq2));
}
```

**Input:** `checkanagrams("JanakAvhad", "AvhadJanak")`\
**Output:** `true`

Anagrams have the same character counts, regardless of order.
`Map.equals()` compares key-value mappings, not insertion order. This
version ignores whitespace and case, but it does not remove punctuation.

## 8. Find the longest word in a sentence

``` java
private static void longestword(String sentence) {
    Optional<String> answer = Arrays.stream(sentence.trim().split("\\s+"))
        .filter(word -> !word.isEmpty())
        .max(Comparator.comparingInt(String::length));

    System.out.println(answer.orElse(null));
}
```

**Output:** `country`

`Comparator.comparingInt(String::length)` compares words by length, and
`max()` selects the longest. The `Optional` covers input with no words.
If multiple words tie, define a tie-breaker if the exact returned word
matters.

## 9. Find the shortest word in a sentence

``` java
private static void shortstrin(String sentence) {
    Optional<String> answer = Arrays.stream(sentence.trim().split("\\s+"))
        .filter(word -> !word.isEmpty())
        .min(Comparator.comparingInt(String::length));

    System.out.println(answer.orElse(null));
}
```

**Output:** `is`

This is the same pipeline as Question 8 except it uses `min()` instead
of `max()`. Both operations return an `Optional` because the stream may
be empty.

## 10. Group words by their first character

``` java
private static void groupalistwords(String sentence) {
    List<String> words = Arrays.stream(sentence.trim().split("\\s+"))
        .filter(word -> !word.isEmpty())
        .toList();

    Map<Character, List<String>> grouped = words.stream()
        .collect(Collectors.groupingBy(
            word -> word.charAt(0), LinkedHashMap::new, Collectors.toList()
        ));

    System.out.println(grouped);
}
```

**Output:**

``` text
{B=[Bharat], i=[is, iss], m=[my, maiyy], c=[ccouct, country]}
```

The classifier `word -> word.charAt(0)` selects the grouping key.
`Collectors.toList()` collects words in each group. Grouping is
case-sensitive, so uppercase and lowercase initials are separate keys.

## 11. Group words by their length

``` java
private static void eachlength(String sentence) {
    List<String> words = Arrays.stream(sentence.trim().split("\\s+"))
        .filter(word -> !word.isEmpty())
        .toList();

    Map<Integer, List<String>> grouped = words.stream()
        .collect(Collectors.groupingBy(
            String::length, LinkedHashMap::new, Collectors.toList()
        ));

    System.out.println(grouped);
}
```

**Output:**

``` text
{6=[Bharat, ccouct], 2=[is, my], 3=[iss], 5=[maiyy], 7=[country]}
```

`String::length` supplies the grouping key. Because a `LinkedHashMap` is
used, keys appear in the order their lengths are first encountered---not
in numeric order. Use a `TreeMap` supplier if sorted length keys are
required.

## Quick comparison

  --------------------------------------------------------------------------
  Problem                             Main operation
  ----------------------------------- --------------------------------------
  Count each character                `groupingBy()` + `counting()`

  First non-repeating character       Frequency map + `findFirst()`

  First repeating character           Frequency map + `filter()` +
                                      `findFirst()`

  Unique duplicate characters         `filter()` + `distinct()`

  Highest-frequency character         `max()` + entry filtering

  Anagram check                       Frequency maps + `Map.equals()`

  Longest / shortest word             `max()` / `min()` + `Comparator`

  Group by first character            `groupingBy(word -> word.charAt(0))`

  Group by length                     `groupingBy(String::length)`
  --------------------------------------------------------------------------

## Common mistakes

1.  **Ignoring case sensitivity:** `A` and `a` are different unless you
    normalize the input.
2.  **Mixing up repeating-character definitions:** the first character
    whose total count exceeds one is not necessarily the character at
    the position of its second occurrence.
3.  **Forgetting `distinct()`:** duplicate characters can appear
    multiple times in the result.
4.  **Assuming keys are sorted:** `LinkedHashMap` preserves encounter
    order; it does not sort.
5.  **Ignoring empty input:** handle empty results from `findFirst()`,
    `min()`, and `max()`.
6.  **Assuming whitespace splitting removes punctuation:**
    `split("\\s+")` separates whitespace only.
7.  **Using `toList()` on older Java versions:** `Stream.toList()`
    requires Java 16+. On Java 8--15, use
    `.collect(Collectors.toList())`.

## Interview practice questions

Try these without looking at the solutions:

1.  Count the frequency of every word in a sentence.
2.  Find the first non-repeating character without a frequency map.
3.  Find the most frequent character while ignoring case.
4.  Return duplicate words without repeating them in the result.
5.  Check anagrams while ignoring spaces, punctuation, and case.
6.  Return every word tied for the longest length.
7.  Group words by their last character.
8.  Group words by length and sort each group alphabetically.
9.  Find the repeating character based on the position of its second
    occurrence.
10. Explain the difference between `HashMap`, `LinkedHashMap`, and
    `TreeMap`.

## Complete runnable example

Save the practice class as `LevelThree.java` inside the `Fundamentals`
package. Keep the helper methods in the same class and uncomment one
call at a time in `main()`.

``` java
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;
```

The examples use `Stream.toList()`, available from Java 16 onward. For
Java 8--15, replace each `.toList()` with
`.collect(Collectors.toList())`.

## Conclusion

The most reusable pattern in these problems is **count first, then
filter or compare**. Once you understand the frequency-map collector,
you can reuse it for duplicate detection, first-occurrence problems,
ranking, and anagram checks.

For word problems, remember that `trim()` and `split("\\s+")` handle
whitespace, `Comparator.comparingInt(String::length)` compares word
lengths, and `groupingBy()` can group values using any classifier
function.

In interviews, explain not only what the pipeline returns, but also why
the operations are ordered that way, what happens for empty input, and
whether the result must preserve encounter order.
