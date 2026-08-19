# Q32 — What is the time complexity before and after Java 8 collision handling?

## 1. Interview Question

> **What is the time complexity of HashMap before and after Java 8 collision handling?**

This is a follow-up to:

- **Q30 — HashMap Internal Working**
- **Q31 — What happens when two keys have the same `hashCode()`?**

---

# 2. 30-Second Interview Answer

> **"Before Java 8, a heavily-collided HashMap bucket used a linked list, so lookup in that bucket could degrade to O(n). From Java 8, when a bucket becomes heavily collided and the implementation's treeification conditions are met, the bucket can be converted into a Red-Black tree, improving lookup in that bucket to O(log n). With good hash distribution, HashMap still provides expected O(1) get and put performance. So the key improvement in Java 8 was the collision bucket changing from a linked structure to a tree under certain conditions."**

---

# 3. First Understand: Normal Case

Normally HashMap works like:

```text
Key
 ↓
hash
 ↓
bucket
 ↓
small number of entries
 ↓
get/put
```

With a good hash distribution:

```text
Expected get() = O(1)
Expected put() = O(1)
Expected remove() = O(1)
```

So Java 8 did **not** change normal HashMap complexity from O(1) to something else.

The important change is the **worst-case behavior of a heavily-collided bucket**.

---

# 4. Before Java 8

Historically, a collided bucket was represented using a linked list.

Example:

```text
Bucket 5
   ↓
Node A
   ↓
Node B
   ↓
Node C
   ↓
Node D
   ↓
Node E
```

If we search for Node E:

```text
A → compare
B → compare
C → compare
D → compare
E → found
```

As the number of nodes `n` in that bucket grows:

```text
Lookup = O(n)
```

---

# 5. Why Was It O(n)?

Because a linked list has sequential access.

To find an entry:

```text
Start at first node
       ↓
Check node
       ↓
Next node
       ↓
Check node
       ↓
...
```

Worst case:

```text
Need to inspect all n nodes
```

Therefore:

```text
O(n)
```

---

# 6. Java 8 Improvement

Java 8 introduced treeification of heavily-collided buckets.

Conceptually:

```text
Before:

Bucket
  ↓
A → B → C → D → E → F → ...

After treeification:

          D
        /   \
       B     F
      / \   / \
     A   C E   G
```

The tree is a **Red-Black tree**.

---

# 7. Why Red-Black Tree?

A Red-Black tree is a self-balancing binary search tree.

Its height remains logarithmic relative to the number of nodes.

Therefore searching a treeified bucket is approximately:

```text
O(log n)
```

instead of:

```text
O(n)
```

for a long linked chain.

---

# 8. Complexity Comparison

| Situation | Before Java 8 | Java 8+ |
|---|---:|---:|
| Good hash distribution | Expected O(1) | Expected O(1) |
| Collision bucket | O(n) | O(n) while represented as a linked structure |
| Treeified bucket | Not applicable | O(log n) |
| Resize | O(n) | O(n) |

The most important row for interviews is:

```text
Linked collision bucket → O(n)
Treeified collision bucket → O(log n)
```

---

# 9. Very Important: Don't Say "Java 8 HashMap Is O(log n)"

This is a common interview mistake.

Wrong:

> **"After Java 8 HashMap lookup is O(log n)."**

Why wrong?

Because most normal lookups do not happen in a treeified bucket.

Correct:

> **"HashMap has expected O(1) lookup with good hash distribution. If a bucket becomes heavily collided and is treeified, lookup in that bucket becomes O(log n)."**

---

# 10. Treeification Does Not Happen Immediately

Another important interview trap.

Do not memorize only:

```text
8 collisions → tree
```

That is incomplete.

In common OpenJDK implementations:

```text
TREEIFY_THRESHOLD = 8
MIN_TREEIFY_CAPACITY = 64
```

The treeification threshold is only one part of the decision.

If the table capacity is too small, HashMap can resize instead of treeifying.

---

# 11. Why Resize Instead of Treeify?

Suppose:

```text
Small table
   ↓
Many entries in one bucket
   ↓
Collision
```

Increasing table capacity can spread entries across more buckets.

Conceptually:

```text
Before:

Bucket 5
A → B → C → D → E

After resize:

Bucket 5 → A → C
Bucket 21 → B → D → E
```

The exact placement depends on the hashes and new capacity.

The key idea is:

> **A larger table can reduce collisions, so HashMap may resize before treeifying when the table is small.**

---

# 12. Complexity of `get()`

### Normal case

```text
hash
 ↓
bucket
 ↓
small number of nodes
```

Expected:

```text
O(1)
```

### Long linked collision bucket

```text
A → B → C → D → E → ...
```

Worst case:

```text
O(n)
```

### Treeified bucket

```text
      D
    /   \
   B     F
```

Approximately:

```text
O(log n)
```

---

# 13. Complexity of `put()`

For `put(key, value)`:

```text
hash
 ↓
bucket
 ↓
find matching key / insertion location
 ↓
insert or replace
```

Expected:

```text
O(1)
```

For a long linked collision bucket:

```text
O(n)
```

For a treeified bucket:

```text
O(log n)
```

But if `put()` triggers a resize, the resize operation itself can require work proportional to the number of entries being moved/repositioned.

---

# 14. Complexity of `remove()`

Similar idea:

```text
Expected      → O(1)
Linked bucket → O(n)
Tree bucket   → O(log n)
```

The exact implementation details of tree deletion and untreeification should not be over-simplified, but this is the useful interview-level model.

---

# 15. What Exactly Changed in Java 8?

### Before Java 8

```text
Collision
   ↓
Linked List
   ↓
Worst-case bucket lookup O(n)
```

### Java 8+

```text
Collision
   ↓
Linked structure initially
   ↓
Heavy collision + conditions met
   ↓
Red-Black Tree
   ↓
Bucket lookup O(log n)
```

That is the core answer.

---

# 16. Why Not Use Tree for Every Bucket?

Because a tree has more overhead than a simple linked structure.

Most HashMap buckets are small.

Using a tree everywhere would add:

```text
More memory
More pointer/structure overhead
More implementation complexity
```

So HashMap uses a simple structure for normal buckets and treeifies only when collisions become significant and conditions justify it.

---

# 17. Why Good `hashCode()` Still Matters

Treeification is a safety mechanism, not an excuse for a bad hash function.

Bad hash:

```java
@Override
public int hashCode() {
    return 1;
}
```

Result:

```text
Many keys
   ↓
Same hash
   ↓
Same bucket
   ↓
Heavy collisions
```

Even if treeification helps, this is still poor design because the hash distribution is terrible.

---

# 18. Why Average Complexity Remains O(1)

Imagine:

```text
1,000,000 keys
```

but a good hash distribution spreads them across many buckets.

Most buckets may contain:

```text
0–few entries
```

So HashMap normally does not scan millions of entries.

Therefore:

```text
Expected lookup ≈ O(1)
```

The collision handling mainly protects against unusually large bucket chains.

---

# 19. Worst Case vs Average Case

This distinction is very important in interviews.

### Average / expected

```text
get()    → O(1)
put()    → O(1)
remove() → O(1)
```

assuming good hash distribution.

### Heavy collision, linked structure

```text
O(n)
```

### Heavy collision, treeified structure

```text
O(log n)
```

So never answer with one complexity without explaining the conditions.

---

# 20. Example

Suppose one bucket contains:

```text
100 entries
```

### Linked list

Searching near the end could require roughly:

```text
100 comparisons
```

Conceptually:

```text
O(100) = O(n)
```

### Balanced tree

A balanced tree can locate an element in logarithmic height.

Conceptually:

```text
log2(100) ≈ 7
```

So the search space is dramatically smaller.

This is why treeification helps under heavy collision.

---

# 21. But Is It Exactly `log2(n)`?

For interview purposes:

```text
O(log n)
```

is enough.

Don't claim every lookup performs exactly `log2(n)` comparisons.

Actual comparisons depend on the tree structure and key/hash comparison behavior.

---

# 22. Important Nuance About Tree Nodes

HashMap's tree nodes still belong to the bucket structure.

Treeification does not mean:

```text
HashMap becomes TreeMap ❌
```

It means:

```text
One heavily-collided bucket
        ↓
Tree-based collision structure
```

The overall collection is still a `HashMap`.

---

# 23. HashMap vs TreeMap

Do not confuse them.

### HashMap

```text
Hash table
 ↓
Expected O(1)
```

### TreeMap

```text
Tree-based sorted map
 ↓
O(log n)
```

Java 8 HashMap treeification does not turn HashMap into TreeMap.

---

# 24. What About Java 7?

For interview shorthand:

```text
Java 7 and earlier
→ collided buckets were linked structures
→ worst-case bucket lookup O(n)
```

Java 8 introduced treeification for heavily-collided buckets.

The exact JDK implementation history can contain additional details, but this is the standard interview comparison.

---

# 25. Resize Complexity

HashMap resizing is another common follow-up.

Suppose:

```text
Capacity = 16
```

and it grows to:

```text
Capacity = 32
```

HashMap has to process existing entries as part of resizing.

Therefore a resize operation can be:

```text
O(n)
```

where `n` is the number of entries involved.

But resize is not performed on every `put()`.

That is why we usually discuss `put()` as expected O(1) amortized under normal assumptions.

---

# 26. Amortized O(1) for `put()`

Most insertions:

```text
O(1)
```

Occasionally:

```text
put()
 ↓
resize
 ↓
O(n)
```

But because resize happens only occasionally as the table grows, the average/amortized insertion cost is treated as O(1) under typical assumptions.

This is a strong senior-level point.

---

# 27. Interview Scenario

### Interviewer:

> Why did Java 8 change HashMap collision handling?

Answer:

> **"To improve worst-case behavior when many keys land in the same bucket. A linked collision chain can take O(n) lookup, while a treeified bucket can provide O(log n) lookup."**

---

# 28. Interview Scenario

### Interviewer:

> Is HashMap faster after Java 8 in every case?

Answer:

> **"Not necessarily. Normal well-distributed lookups were already expected O(1). The major improvement is protection against heavily-collided buckets, where the bucket can be treeified."**

---

# 29. Interview Scenario

### Interviewer:

> What is the worst-case lookup complexity in Java 8+ HashMap?

Interview-safe answer:

> **"For a treeified collision bucket, lookup is O(log n). If you're discussing the general HashMap behavior, expected lookup is O(1) with good hash distribution. The exact worst-case discussion should account for implementation details and pathological inputs."**

For standard interview discussion, the key comparison is:

```text
Before treeification → O(n)
After treeification  → O(log n)
```

---

# 30. Interview Scenario

### Interviewer:

> If there are 8 collisions, will Java 8 always treeify?

Answer:

> **"No. The treeification threshold is not the only condition. The table capacity also matters. In common OpenJDK implementations, if capacity is below the minimum treeification capacity, HashMap prefers resizing instead of treeifying."**

---

# 31. Complete Visual

```text
                 HashMap
                    ↓
              Calculate bucket
                    ↓
             How many collisions?
                    ↓
          ┌─────────┴─────────┐
          │                   │
       Normal              Heavy
          │              collision
          ↓                   ↓
      Expected O(1)      Check conditions
                              ↓
                     ┌────────┴────────┐
                     │                 │
                  Resize          Treeify
                     │                 │
                     ↓                 ↓
                 Reduce          Red-Black
                 collisions          Tree
                                       ↓
                                   O(log n)
```

---

# 32. Quick Comparison Table

| Topic | Before Java 8 | Java 8+ |
|---|---|---|
| Normal lookup | Expected O(1) | Expected O(1) |
| Collision structure | Linked list | Linked structure initially |
| Heavy collision | O(n) bucket lookup | Can treeify |
| Treeified bucket | Not available | O(log n) bucket lookup |
| Resize | O(n) | O(n) |
| Main improvement | — | Better heavy-collision behavior |

---

# 33. 2-Minute Interview Answer

> **"HashMap normally provides expected O(1) get, put and remove when keys are distributed well across buckets. Before Java 8, if many keys collided into the same bucket, the bucket was represented as a linked structure, so lookup in that bucket could degrade to O(n). Java 8 introduced treeification for heavily-collided buckets. When the relevant thresholds and capacity conditions are satisfied, the bucket can be converted into a Red-Black tree, improving lookup in that bucket to O(log n). This does not mean HashMap became O(log n) overall; normal lookups remain expected O(1). Also, treeification does not necessarily happen at exactly eight entries because table capacity is another condition. Resize operations themselves can take O(n), while normal insertion remains amortized/expected O(1)."**

---

# 34. 30-Second Hinglish Answer

> **"Java 8 se pehle agar HashMap ke ek bucket mein bahut collisions hote the, to linked list ki wajah se lookup O(n) tak degrade ho sakta tha. Java 8 mein heavily-collided bucket ko conditions satisfy hone par Red-Black tree mein convert kiya ja sakta hai, jisse us bucket ka lookup O(log n) ho jata hai. Lekin normal HashMap lookup ab bhi expected O(1) hota hai. Aur sirf 8 entries hone se immediately tree nahi banta; table capacity bhi matter karti hai."**

---

# 35. Memory Trick

## **Before → List → O(n)**

## **Java 8+ → Tree → O(log n)**

And remember:

```text
Normal HashMap → Expected O(1)
```

One-line memory:

> **"Normal case O(1), collision list O(n), treeified collision O(log n)."**

---

# 36. Common Interview Mistakes

### ❌ Mistake 1

> Java 8 changed HashMap from O(1) to O(log n).

### ✅ Correct

Java 8 improved heavily-collided bucket behavior.

---

### ❌ Mistake 2

> Every 8th collision creates a tree.

### ✅ Correct

Treeification depends on threshold **and** table capacity/implementation conditions.

---

### ❌ Mistake 3

> HashMap always has O(1) complexity.

### ✅ Correct

Expected O(1) with good hash distribution; collision behavior can differ.

---

### ❌ Mistake 4

> Java 8 HashMap uses TreeMap internally.

### ✅ Correct

It can use a Red-Black tree for a heavily-collided bucket. It does not become a TreeMap.

---

### ❌ Mistake 5

> Resize is O(1).

### ✅ Correct

A resize operation can require O(n) work, although resize is infrequent and normal insertion is amortized/expected O(1).

---

# 37. Follow-Up Questions

- Why did Java 8 introduce treeification?
- What is `TREEIFY_THRESHOLD`?
- What is `MIN_TREEIFY_CAPACITY`?
- Why does HashMap resize instead of treeify for a small table?
- Why is a good `hashCode()` important?
- What is the difference between HashMap and TreeMap?
- What is amortized O(1)?
- What happens during HashMap resize?
- What is the worst-case complexity of HashMap?
- How does `ConcurrentHashMap` handle collisions?

---

## Status

✅ **Q32 Solution Completed**

Previous: [Q31 — Same hashCode / Collision](../Q31-Same-HashCode-Collision/README.md)

Next: **Q33 — Coding: Find the city with the highest number of repeated characters.**
