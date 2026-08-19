# Q30 — Explain HashMap Internal Working

> **Interview Question:** Explain HashMap internal working.

## 1. 30-Second Interview Answer

> **"Java HashMap stores key-value pairs using a hash table. Internally it maintains an array of buckets. When we call `put(key, value)`, HashMap calculates the key's hash, applies a hash-spreading function, calculates the bucket index, and stores the entry in that bucket. If multiple keys map to the same bucket, collision is handled using a linked structure, and in modern Java, a heavily-collided bucket can be treeified into a Red-Black tree. During `get()`, HashMap calculates the same bucket and then compares the hash and keys using `equals()` to find the value. Average lookup is O(1), while a heavily-collided tree bucket can provide O(log n) lookup."**

---

# 2. HashMap Kya Hai?

`HashMap<K,V>` Java ka collection hai jo:

```text
Key → Value
```

store karta hai.

Example:

```java
Map<Integer, String> map = new HashMap<>();

map.put(101, "Nirbhay");
map.put(102, "Rahul");
map.put(103, "Amit");
```

Conceptually:

```text
101 → Nirbhay
102 → Rahul
103 → Amit
```

Important properties:

- Key unique hoti hai.
- One `null` key allowed.
- Multiple `null` values allowed.
- HashMap is not synchronized.
- Average `put/get/remove` is O(1), assuming good hash distribution.

---

# 3. Internal Structure

Conceptually:

```text
HashMap
   |
   └── table[]
        |
        ├── bucket 0
        ├── bucket 1
        ├── bucket 2
        ├── bucket 3
        └── ...
```

Modern Java implementations use an internal array of nodes/entries.

Conceptually a node contains:

```text
Node
 ├── hash
 ├── key
 ├── value
 └── next
```

For a treeified bucket, the nodes are represented as tree nodes rather than a simple linked chain.

---

# 4. HashMap `put()` — Step by Step

Suppose:

```java
map.put("Java", 100);
```

Flow:

```text
key = "Java"
     ↓
key.hashCode()
     ↓
hash spreading
     ↓
bucket index calculation
     ↓
find bucket
     ↓
insert / replace
```

---

# 5. Step 1 — Calculate hashCode()

Java calls:

```java
key.hashCode()
```

Example conceptually:

```text
"Java"
  ↓
hashCode()
  ↓
integer hash
```

The exact integer depends on the key's implementation.

Important:

> `hashCode()` does **not** directly mean bucket index.

---

# 6. Step 2 — Hash Spreading

HashMap applies a hash-spreading operation so that useful bits from the hash are mixed before bucket selection.

Conceptually, modern Java implementations use a transformation equivalent to:

```java
h ^ (h >>> 16)
```

Why?

Because bucket selection commonly uses low-order bits. Mixing high bits into lower bits can improve distribution.

---

# 7. Step 3 — Calculate Bucket Index

For a table of size `n`, modern HashMap uses a power-of-two table and computes the bucket index conceptually as:

```java
index = (n - 1) & hash
```

Example:

```text
n = 16
hash = ...

index = (16 - 1) & hash
      = 15 & hash
```

This is efficient compared with `% n` and works correctly because the table size is maintained as a power of two.

---

# 8. Why Power of Two?

HashMap table capacity is maintained as a power of two.

Examples:

```text
16
32
64
128
256
...
```

This allows:

```java
(n - 1) & hash
```

to efficiently select the bucket.

---

# 9. Empty Bucket

If calculated bucket is empty:

```text
Bucket 5
   ↓
null
```

HashMap creates a node:

```text
Bucket 5
   ↓
[hash | key | value | next=null]
```

Done.

---

# 10. Collision Kya Hai?

Suppose two different keys produce the same bucket index.

```text
Key A → bucket 5
Key B → bucket 5
```

This is called a **collision**.

Important:

> Different keys can have the same hash code or different hash codes can still map to the same bucket.

Collision is therefore possible even when `hashCode()` values differ.

---

# 11. Collision Handling — Linked Structure

Initially a collided bucket can contain a linked chain:

```text
Bucket 5
   ↓
Node A
   ↓
Node B
   ↓
Node C
```

HashMap traverses nodes and checks whether the key matches.

Conceptually:

```text
same bucket
   ↓
compare hash
   ↓
compare key
   ↓
found?
```

---

# 12. How Does HashMap Know It Is the Same Key?

For a matching entry, HashMap effectively checks the hash and key equality.

Conceptually:

```java
existingHash == newHash
&&
(existingKey == newKey || newKey.equals(existingKey))
```

So:

```text
hashCode() → narrows candidates
equals()   → confirms logical key equality
```

---

# 13. Same Key → Replace Value

Example:

```java
map.put("Java", 100);
map.put("Java", 200);
```

Result:

```text
Java → 200
```

It does not create a second logical mapping for the same key.

---

# 14. Different Keys → Both Can Exist

Example:

```java
map.put("A", 100);
map.put("B", 200);
```

Even if both land in the same bucket:

```text
Bucket X
  ↓
A → 100
  ↓
B → 200
```

They can coexist because their keys are not equal.

---

# 15. Java 8+ Treeification

A long collision chain can make lookup slow.

To improve worst-case behavior, modern HashMap implementations can convert a heavily-collided bucket from a linked structure into a **Red-Black tree**, subject to implementation thresholds and capacity conditions.

Conceptually:

```text
Linked chain

A
↓
B
↓
C
↓
D
↓
...

       ↓ treeify

Red-Black Tree
```

---

# 16. Treeification Threshold

In common OpenJDK implementations, the treeification threshold is:

```text
TREEIFY_THRESHOLD = 8
```

But treeification also depends on table capacity.

A commonly cited minimum capacity is:

```text
MIN_TREEIFY_CAPACITY = 64
```

If the table is too small, HashMap may prefer resizing rather than treeifying.

Important interview line:

> **"Treeification does not simply mean the 8th node always becomes a tree; HashMap also considers the table capacity."**

---

# 17. Why Treeify?

Linked list lookup:

```text
O(n)
```

Red-Black tree lookup:

```text
O(log n)
```

So heavy collision cases become much more manageable.

But average HashMap performance remains:

```text
O(1)
```

under good hash distribution.

---

# 18. `get()` Internal Working

Suppose:

```java
map.get("Java");
```

Flow:

```text
"Java"
   ↓
hashCode()
   ↓
hash spreading
   ↓
bucket index
   ↓
find bucket
   ↓
compare candidate node(s)
   ↓
return value
```

---

# 19. `get()` with Linked Chain

```text
Bucket 5
   ↓
Node A
   ↓
Node B
   ↓
Node C
```

HashMap checks candidates until it finds matching key.

```text
hash match?
   ↓ yes
key equals?
   ↓ yes
return value
```

---

# 20. `get()` with Treeified Bucket

If bucket is a Red-Black tree:

```text
          Node B
         /      \
      Node A    Node D
                  \
                  Node E
```

HashMap uses tree navigation/comparisons to locate the matching entry.

Conceptual complexity:

```text
O(log n)
```

for the tree bucket.

---

# 21. `remove()` Internal Working

For:

```java
map.remove("Java");
```

HashMap:

```text
hashCode()
   ↓
hash spreading
   ↓
bucket index
   ↓
find matching node
   ↓
remove node
```

If it is a linked chain, links are adjusted.

If it is a tree bucket, tree structure is updated according to the implementation.

---

# 22. Resize — Very Important

HashMap has:

```text
capacity
load factor
threshold
```

Default constructor behavior in modern JDKs uses lazy table allocation; the commonly discussed default load factor is:

```text
0.75
```

The table is resized when the number of entries crosses the resize threshold.

Conceptually:

```text
threshold = capacity × load factor
```

For example:

```text
capacity = 16
load factor = 0.75

threshold = 12
```

When size grows beyond the threshold, HashMap resizes.

---

# 23. What Happens During Resize?

Suppose:

```text
Capacity = 16
```

Resize:

```text
16 → 32
```

HashMap allocates a larger table and redistributes entries according to the new capacity.

Because capacity doubles, the new bucket placement can change.

---

# 24. Why Load Factor 0.75?

Trade-off between:

```text
Memory
   ↕
Collision probability
```

Lower load factor:

```text
More memory
Fewer collisions
```

Higher load factor:

```text
Less memory
Potentially more collisions
```

`0.75` is a commonly used default balancing memory and performance.

---

# 25. Initial Capacity vs Threshold

Interview trap:

> "HashMap creates 16 buckets immediately when you call `new HashMap<>()`."

Not necessarily in modern JDK implementations.

The default table can be allocated lazily when entries are first inserted.

For interview purposes:

```text
Default initial capacity concept = 16
Default load factor = 0.75
```

But be precise if asked about actual allocation timing.

---

# 26. `hashCode()` and `equals()` Contract

Very important.

If:

```java
a.equals(b) == true
```

then:

```java
a.hashCode() == b.hashCode()
```

must also be true.

But reverse is not required:

```text
same hashCode
≠
objects are equal
```

This is why HashMap still needs `equals()` after narrowing candidates using hash information.

---

# 27. Custom Key Example

```java
class Employee {
    private int id;
    private String name;

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    @Override
    public boolean equals(Object obj) {
        // logical equality implementation
    }
}
```

If a key is mutable and fields used by `hashCode()`/`equals()` change after insertion, lookup can fail unexpectedly.

---

# 28. Why Mutable Keys Are Dangerous

Example:

```java
Map<Employee, String> map = new HashMap<>();
```

If `Employee.id` participates in `hashCode()` and changes after insertion:

```text
Old hash → Bucket 5
       ↓
ID changed
       ↓
New hash → Bucket 10
```

Then:

```java
map.get(employee)
```

may search the new bucket and fail to find the entry stored in the old bucket.

Best practice:

> Use immutable key state.

---

# 29. Null Key

HashMap allows one `null` key:

```java
map.put(null, "value");
```

and multiple null values:

```java
map.put(1, null);
map.put(2, null);
```

The `null` key is handled specially by the implementation rather than by calling `hashCode()` on null.

---

# 30. Why HashMap Is Not Thread-Safe

Multiple threads modifying a regular HashMap concurrently can lead to data races and incorrect behavior.

For concurrent access, consider:

```java
ConcurrentHashMap
```

rather than externally assuming HashMap is safe.

---

# 31. HashMap vs ConcurrentHashMap

| Feature | HashMap | ConcurrentHashMap |
|---|---|---|
| Thread-safe | No | Yes, for its supported concurrent operations |
| Null key | One allowed | Not allowed |
| Null values | Allowed | Not allowed |
| Concurrent access | Not designed for it | Designed for concurrent access |
| Typical use | Single-threaded / externally synchronized contexts | Multi-threaded shared map |

Important:

> `ConcurrentHashMap` is not simply a "synchronized HashMap"; it is designed for concurrent access with finer-grained coordination and non-blocking reads in many cases.

---

# 32. Complexity

Average case:

```text
put()    → O(1)
get()    → O(1)
remove() → O(1)
```

Heavy collision linked bucket:

```text
O(n)
```

Treeified bucket:

```text
O(log n)
```

Resize:

```text
O(n)
```

because entries need to be redistributed.

Important:

> These are conceptual/expected complexities, not guarantees for every operation under every implementation detail.

---

# 33. Complete `put()` Flow

Memorize this:

```text
put(key, value)
      ↓
hashCode()
      ↓
hash spreading
      ↓
calculate bucket index
      ↓
bucket empty?
   ↙           ↘
 yes            no
 ↓               ↓
insert       compare hash/key
                 ↓
          same key?
          ↙       ↘
        yes        no
         ↓          ↓
   replace value   collision
                       ↓
                 linked/tree structure
```

---

# 34. Complete `get()` Flow

```text
get(key)
   ↓
hashCode()
   ↓
hash spreading
   ↓
bucket index
   ↓
find bucket
   ↓
compare hash + key
   ↓
return value
```

---

# 35. Interview Scenario

### Interviewer:

> Suppose two different keys have the same hash code. What happens?

Answer:

```text
Same hash
  ↓
Same bucket
  ↓
Collision
  ↓
HashMap stores both entries in the bucket
  ↓
Uses equals() to distinguish logical keys
```

If the bucket becomes heavily collided and capacity conditions allow it, it can be treeified.

---

# 36. Interview Scenario — Why O(1)?

### Interviewer:

> Why is HashMap get() O(1)?

Answer:

> Because the hash lets HashMap directly calculate the bucket instead of scanning the complete collection. With a well-distributed hash function, only a small number of candidates need to be examined, giving expected O(1) lookup.

Don't say:

> "HashMap always gives O(1)."

Say:

> **"Expected/average O(1) under good hash distribution."**

---

# 37. Interview Scenario — Java 8 Improvement

### Question:

> What improvement happened in Java 8 for HashMap collisions?

Answer:

> **"When a bucket becomes heavily collided, HashMap can convert the linked structure into a Red-Black tree, subject to thresholds and table capacity. This improves lookup in that bucket from linear behavior toward O(log n)."**

---

# 38. Senior-Level Detail — Resize vs Treeify

If collision count reaches the treeification threshold but the table is still small:

```text
Don't immediately treeify
        ↓
Resize table instead
```

Why?

A larger table can spread entries across more buckets and reduce collisions without maintaining tree structures.

This demonstrates that you understand the implementation rather than memorizing `8 → tree`.

---

# 39. Senior-Level Detail — HashMap Is Not Just an Array

A weak answer:

> "HashMap is an array."

Better answer:

> **"HashMap uses an array of buckets, with each bucket containing nodes that can form a linked structure and, under heavy collisions, a Red-Black tree."**

---

# 40. Senior-Level Detail — Why `equals()` Matters

Weak answer:

> "HashMap uses hashCode to find the value."

Incomplete.

Correct mental model:

```text
hashCode
   ↓
find bucket
   ↓
hash comparison
   ↓
equals()
   ↓
identify exact logical key
```

---

# 41. Full Example

```java
Map<String, Integer> map = new HashMap<>();

map.put("A", 10);
map.put("B", 20);
map.put("C", 30);

Integer value = map.get("B");
```

Conceptually:

```text
put("B", 20)
      ↓
"B".hashCode()
      ↓
spread hash
      ↓
calculate bucket
      ↓
store Node("B", 20)
```

Then:

```text
get("B")
      ↓
"B".hashCode()
      ↓
same hash calculation
      ↓
same bucket
      ↓
compare key
      ↓
20
```

---

# 42. What Happens If `hashCode()` Is Bad?

Suppose many different keys return the same hash:

```java
@Override
public int hashCode() {
    return 1;
}
```

Then:

```text
Many keys
   ↓
Same hash
   ↓
Same bucket
   ↓
Many collisions
```

Performance can degrade significantly.

So good hash distribution matters.

---

# 43. What If `equals()` Is Wrong?

If logically equal objects are not considered equal:

```text
Expected same key
        ↓
Wrong equals()
        ↓
HashMap treats them as different
```

This can create unexpected duplicate logical entries.

So both methods must follow the contract.

---

# 44. Most Important Interview Points

Remember these 10:

```text
1. HashMap stores key-value pairs.
2. Internally it uses bucket array + nodes.
3. hashCode() is used to derive hash information.
4. Hash spreading improves distribution.
5. Bucket index is based on hash and table capacity.
6. Collision is handled within the bucket.
7. equals() identifies the exact logical key.
8. Heavy collisions can trigger Red-Black tree conversion.
9. Resize happens when threshold is exceeded.
10. Average get/put is O(1), not guaranteed O(1).
```

---

# 45. 2-Minute Interview Answer

> **"HashMap internally works like a hash table. It maintains an array of buckets. When we insert a key-value pair, HashMap first gets the key's hashCode, applies hash spreading, and calculates a bucket index using the table capacity. If the bucket is empty, it stores a node there. If another entry is already present, we have a collision. HashMap compares the hash and then uses equals() to determine whether it is the same logical key. If it is the same key, the value is replaced; otherwise the new entry is added to the bucket's collision structure. In modern Java, a heavily-collided bucket can be converted into a Red-Black tree when the implementation's treeification conditions are met, improving lookup in that bucket from linear to logarithmic behavior. HashMap also resizes when its threshold, based on capacity and load factor, is exceeded. The average complexity of get, put and remove is O(1) with good hash distribution, while heavy collision cases can be O(n) in a linked structure or O(log n) in a treeified bucket."**

---

# 46. 30-Second Hinglish Answer

> **"HashMap internally ek bucket array use karta hai. `put()` ke time key ka `hashCode()` liya jata hai, hash spread kiya jata hai aur bucket index calculate hota hai. Agar bucket empty hai to entry store hoti hai; agar same bucket mein entry hai to collision hota hai. HashMap hash aur `equals()` ke through exact key identify karta hai. Heavy collision mein modern Java HashMap bucket ko Red-Black tree mein treeify kar sakta hai, subject to capacity/threshold conditions. Load factor ke basis par resize bhi hota hai. Average `get/put` O(1) hota hai, but collision ke case mein linked structure O(n) aur treeified bucket O(log n) tak ho sakta hai."**

---

# 47. Memory Trick

### **H → B → C → E → T → R**

```text
Hash
 ↓
Bucket
 ↓
Collision
 ↓
Equals
 ↓
Tree
 ↓
Resize
```

One-line memory:

> **"Hash → Bucket → Collision → Equals → Tree → Resize."**

---

# 48. Common Interview Mistakes

### ❌ Mistake 1

> HashMap uses only hashCode.

### ✅ Correct

```text
hashCode → bucket candidate
equals   → exact key
```

### ❌ Mistake 2

> 8 entries means tree for sure.

### ✅ Correct

Treeification also depends on table capacity and implementation conditions.

### ❌ Mistake 3

> HashMap always gives O(1).

### ✅ Correct

Expected O(1); collisions can degrade performance.

### ❌ Mistake 4

> HashMap creates 16 buckets immediately on construction in every modern JDK.

### ✅ Correct

Table allocation can be lazy in modern implementations.

### ❌ Mistake 5

> Same hash means same key.

### ✅ Correct

```text
Same hash ≠ same key
```

`equals()` still matters.

---

# 49. Follow-Up Questions

### Q31. What happens when two keys have the same hashCode?

Next question in this interview series.

### Q32. What is the time complexity before and after Java 8 collision handling?

Next question in this interview series.

### Other likely follow-ups

- Why does HashMap use power-of-two capacity?
- What is load factor?
- What happens during resize?
- Why is `equals()` required?
- Why are mutable keys dangerous?
- Difference between HashMap and ConcurrentHashMap?
- How does HashMap handle null keys?
- Why is HashMap not thread-safe?
- What happens if `hashCode()` always returns the same value?

---

## Status

✅ **Q30 Solution Completed**

Next: **Q31 — What happens when two keys have the same hashCode?**
