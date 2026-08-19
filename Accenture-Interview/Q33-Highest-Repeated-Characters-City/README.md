# Q33 — Find the city with the highest number of repeated characters

## 1. Interview Question

> Given a list of city names, return the city that contains the highest number of repeated characters.

Example:

```text
Input:
["Delhi", "Mumbai", "Chennai", "Pune", "Bangalore"]

Expected Output:
Bangalore
```

---

# 2. First Clarify the Requirement

Interview mein pehle ye clarify karna important hai ki **"highest number of repeated characters"** ka exact meaning kya hai.

For this solution, we define it as:

> **For each city, count the number of characters whose frequency is greater than 1. Return the city with the highest such count.**

Example:

```text
Bangalore
```

Character frequencies:

```text
b → 1
a → 2
n → 2
g → 2
l → 1
o → 1
r → 1
e → 1
```

Repeated character types:

```text
a, n, g
```

Count = `3`.

So Bangalore has 3 repeated-character types.

> **Important:** If the interviewer means "total duplicate occurrences" instead, the counting logic changes slightly. Always clarify this ambiguity.

---

# 3. Simple Approach

For every city:

```text
City
 ↓
Count character frequencies
 ↓
Find characters with frequency > 1
 ↓
Count them
 ↓
Compare with current maximum
```

Example:

```text
Chennai
```

```text
c → 1
h → 1
e → 2
n → 2
a → 1
i → 1
```

Repeated character types:

```text
e, n
```

Count = `2`.

---

# 4. Java Solution — HashMap

```java
import java.util.*;

public class HighestRepeatedCharacters {

    public static String findCity(String[] cities) {
        String result = null;
        int maxRepeated = 0;

        for (String city : cities) {
            Map<Character, Integer> frequency = new HashMap<>();

            for (char ch : city.toLowerCase().toCharArray()) {
                frequency.put(ch, frequency.getOrDefault(ch, 0) + 1);
            }

            int repeatedCount = 0;

            for (int count : frequency.values()) {
                if (count > 1) {
                    repeatedCount++;
                }
            }

            if (repeatedCount > maxRepeated) {
                maxRepeated = repeatedCount;
                result = city;
            }
        }

        return result;
    }

    public static void main(String[] args) {
        String[] cities = {
            "Delhi",
            "Mumbai",
            "Chennai",
            "Pune",
            "Bangalore"
        };

        System.out.println(findCity(cities));
    }
}
```

Output:

```text
Bangalore
```

---

# 5. Dry Run

Input:

```text
[Delhi, Mumbai, Chennai, Pune, Bangalore]
```

## Delhi

```text
d → 1
e → 1
l → 1
h → 1
i → 1
```

Repeated types = `0`

```text
max = 0
result = Delhi? 
```

Our code updates only when `repeatedCount > maxRepeated`, so with zero it keeps `result = null`.

For a cleaner implementation, initialize the result using the first city or use `maxRepeated = -1`.

---

# 6. Interview-Correct Implementation

A cleaner version is:

```java
import java.util.*;

public class HighestRepeatedCharacters {

    public static String findCity(String[] cities) {
        if (cities == null || cities.length == 0) {
            return null;
        }

        String result = cities[0];
        int maxRepeated = -1;

        for (String city : cities) {
            Map<Character, Integer> frequency = new HashMap<>();

            for (char ch : city.toLowerCase().toCharArray()) {
                frequency.put(ch, frequency.getOrDefault(ch, 0) + 1);
            }

            int repeatedCount = 0;

            for (int count : frequency.values()) {
                if (count > 1) {
                    repeatedCount++;
                }
            }

            if (repeatedCount > maxRepeated) {
                maxRepeated = repeatedCount;
                result = city;
            }
        }

        return result;
    }
}
```

Now even if every city has zero repeated characters, the first city is returned.

---

# 7. Step-by-Step Dry Run

For:

```text
Bangalore
```

Frequency map:

```text
b = 1
a = 2
n = 2
g = 2
l = 1
o = 1
r = 1
e = 1
```

Now iterate through values:

```text
1 → ignore
2 → repeatedCount = 1
2 → repeatedCount = 2
2 → repeatedCount = 3
1 → ignore
1 → ignore
1 → ignore
1 → ignore
```

Final:

```text
repeatedCount = 3
```

If this is greater than the current maximum:

```text
result = Bangalore
maxRepeated = 3
```

---

# 8. Why HashMap?

We need to maintain:

```text
character → frequency
```

HashMap gives us a convenient frequency table.

Conceptually:

```text
'a' → 2
'b' → 1
'n' → 2
'g' → 2
```

Then we simply count how many frequencies are greater than `1`.

---

# 9. Alternative — Integer Array

Because English letters are limited, we can avoid HashMap.

If the input is guaranteed to contain only lowercase English letters:

```java
public static String findCity(String[] cities) {
    if (cities == null || cities.length == 0) {
        return null;
    }

    String result = cities[0];
    int maxRepeated = -1;

    for (String city : cities) {
        int[] frequency = new int[26];

        for (char ch : city.toLowerCase().toCharArray()) {
            if (ch >= 'a' && ch <= 'z') {
                frequency[ch - 'a']++;
            }
        }

        int repeatedCount = 0;

        for (int count : frequency) {
            if (count > 1) {
                repeatedCount++;
            }
        }

        if (repeatedCount > maxRepeated) {
            maxRepeated = repeatedCount;
            result = city;
        }
    }

    return result;
}
```

This uses fixed extra space:

```text
O(26) = O(1)
```

under the lowercase-English-letter assumption.

---

# 10. Time Complexity

Let:

```text
N = number of cities
L = maximum length of a city name
```

For each city:

```text
Build frequency map → O(L)
Count frequencies   → O(L) worst case
```

Therefore:

```text
Total Time = O(N × L)
```

More precisely, if each city has length `Li`:

```text
O(Σ Li)
```

which is the total number of characters processed.

---

# 11. Space Complexity

For one city, the frequency map contains at most the number of distinct characters.

If `K` is the number of distinct characters:

```text
Space = O(K)
```

For lowercase English letters with an array:

```text
Space = O(26) = O(1)
```

---

# 12. Important Ambiguity — What Does "Repeated Characters" Mean?

This question can be interpreted in two ways.

## Interpretation A — Repeated character types

Example:

```text
Bangalore
```

```text
a = 2
n = 2
g = 2
```

Repeated character types:

```text
a, n, g
```

Count = `3`.

---

## Interpretation B — Duplicate occurrences

For each character:

```text
a = 2 → one duplicate occurrence
a
n = 2 → one duplicate occurrence
g = 2 → one duplicate occurrence
```

Total duplicate occurrences = `3`.

For frequency `3`:

```text
xxx
```

There are:

```text
frequency - 1 = 2
```

duplicate occurrences.

So:

```text
Repeated character types:
count > 1

Duplicate occurrences:
count - 1
```

This distinction is important in an interview.

---

# 13. If Interviewer Means Highest Duplicate Occurrences

Use:

```java
int duplicateOccurrences = 0;

for (int count : frequency.values()) {
    if (count > 1) {
        duplicateOccurrences += count - 1;
    }
}
```

For:

```text
Bangalore
```

```text
a = 2 → +1
n = 2 → +1
g = 2 → +1
```

Total:

```text
3
```

For a city such as:

```text
Mississippi
```

frequency-based definitions can produce different scores, so clarify the requirement first.

---

# 14. What If Two Cities Have the Same Maximum?

This is another clarification point.

Possible rules:

### Option 1 — Return the first city

Use:

```java
if (repeatedCount > maxRepeated)
```

### Option 2 — Return the last city

Use:

```java
if (repeatedCount >= maxRepeated)
```

### Option 3 — Return all tied cities

Use a list:

```java
List<String> result = new ArrayList<>();
```

Interview mein explicitly state your tie-breaking rule.

---

# 15. Case Sensitivity

Should:

```text
A
```

and:

```text
a
```

be treated as the same character?

Usually for a city-name problem, we can make it case-insensitive:

```java
city.toLowerCase()
```

But in production code, locale-aware lowercasing may be relevant for international text.

For this interview problem, the simple approach is enough unless the interviewer asks about Unicode/locales.

---

# 16. Spaces and Special Characters

If input can contain:

```text
New Delhi
```

decide whether space should count.

Usually:

```text
Ignore spaces and punctuation
```

if the requirement is about letters.

Example:

```java
if (Character.isLetterOrDigit(ch)) {
    frequency.put(ch, frequency.getOrDefault(ch, 0) + 1);
}
```

Again, clarify instead of assuming.

---

# 17. Optimized One-Pass Per City

We can avoid a second pass over `frequency.values()` by updating the repeated-character count while building the frequency map.

```java
public static String findCity(String[] cities) {
    if (cities == null || cities.length == 0) {
        return null;
    }

    String result = cities[0];
    int maxRepeated = -1;

    for (String city : cities) {
        Map<Character, Integer> frequency = new HashMap<>();
        int repeatedCount = 0;

        for (char ch : city.toLowerCase().toCharArray()) {
            int newCount = frequency.getOrDefault(ch, 0) + 1;
            frequency.put(ch, newCount);

            if (newCount == 2) {
                repeatedCount++;
            }
        }

        if (repeatedCount > maxRepeated) {
            maxRepeated = repeatedCount;
            result = city;
        }
    }

    return result;
}
```

Why `newCount == 2`?

Because a character should be counted as a **repeated character type exactly once** when it changes from:

```text
1 occurrence → 2 occurrences
```

If it later becomes:

```text
3
4
5
```

we do not increment the repeated-character-type count again.

---

# 18. Complexity of Optimized Version

Still:

```text
Time = O(total characters)
```

or:

```text
O(Σ Li)
```

Space:

```text
O(K)
```

where `K` is the number of distinct characters.

We mainly removed the second traversal over the frequency map; asymptotic complexity remains the same.

---

# 19. Java Streams Version

For interview coding, a normal loop is usually easier to explain.

But if interviewer asks for Java 8 Stream approach:

```java
Map<Character, Long> frequency = city.toLowerCase()
        .chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()
        ));

long repeatedCount = frequency.values().stream()
        .filter(count -> count > 1)
        .count();
```

Then compare `repeatedCount` for each city.

For an interview, prefer the loop first because it clearly demonstrates algorithmic thinking.

---

# 20. Edge Cases

### Empty list

```text
[]
```

Return:

```text
null
```

or throw an appropriate exception depending on the contract.

### Null input

```text
null
```

Handle explicitly.

### Empty city

```text
""
```

Repeated count = `0`.

### One city

Return that city.

### All cities have zero repeats

Return the first city if that is the defined tie-breaking rule.

### Multiple cities tie

Define whether to return first, last, or all.

---

# 21. Interview Flow

When interviewer gives the problem, explain:

```text
1. Clarify what repeated means
2. Decide case sensitivity
3. Decide tie-breaking
4. Build frequency map per city
5. Count frequencies > 1
6. Track maximum
7. Return city
```

This shows structured problem solving rather than immediately writing code.

---

# 22. 2-Minute Interview Answer

> **"I would iterate through every city and maintain a frequency map of its characters. For each character, I increment its count. After processing the city, I count how many distinct characters have frequency greater than one. I compare that count with the current maximum and keep the corresponding city. For example, Bangalore has a, n and g repeated, so its repeated-character-type count is three. The overall time complexity is O(total number of characters across all cities), or O(N × L) if N is the number of cities and L is the maximum city length. The extra space is O(K), where K is the number of distinct characters. I would also clarify whether repeated means repeated character types or total duplicate occurrences, and what to do in case of a tie."**

---

# 23. 30-Second Hinglish Answer

> **"Main har city ke characters ka frequency map banaunga. Uske baad count karunga ki kitne characters ki frequency 1 se greater hai. Jis city ka ye count maximum hoga, us city ko answer mein rakhunga. Example Bangalore mein a, n aur g repeat hote hain, so repeated character types ka count 3 hai. Time complexity total characters ke according O(ΣLi) hai aur space O(K) hai. Interview mein main pehle clarify karunga ki repeated characters ka matlab repeated character types hai ya total duplicate occurrences, aur tie hone par kya return karna hai."**

---

# 24. Memory Trick

```text
CITY
 ↓
Frequency Map
 ↓
count > 1
 ↓
Repeated Count
 ↓
Compare MAX
 ↓
Return City
```

One-line memory:

> **"Count characters → count repeats → track maximum city."**

---

# 25. Common Interview Mistakes

### ❌ Mistake 1

Counting total characters instead of repeated characters.

### ❌ Mistake 2

Assuming the meaning of "repeated" without clarifying.

### ❌ Mistake 3

Using nested loops to compare every character with every other character unnecessarily.

### ❌ Mistake 4

Forgetting case sensitivity.

### ❌ Mistake 5

Not handling ties.

### ❌ Mistake 6

Not handling empty/null input.

### ❌ Mistake 7

Claiming O(1) space when using a HashMap without explaining the character-set assumption.

---

# 26. Follow-Up Questions

- Can you solve it without HashMap?
- What is the time complexity?
- What is the space complexity?
- What if the input contains Unicode characters?
- What if spaces should be ignored?
- What if two cities have the same maximum?
- Can you return all cities with the maximum repeated count?
- Can you solve it using Java Streams?
- How would you count duplicate occurrences instead of repeated character types?
- Can you optimize it to one pass per city?

---

## Status

✅ **Q33 Solution Completed**

Previous: [Q32 — HashMap Collision Time Complexity](../Q32-HashMap-Collision-Time-Complexity/README.md)

Next: **Continue with the next interview question.**
