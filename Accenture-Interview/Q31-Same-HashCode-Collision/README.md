# Q31 — What happens when two keys have the same `hashCode()`?

## 1. Interview Question

> **What happens when two keys have the same `hashCode()` in HashMap?**

This is a direct follow-up to **Q30 — HashMap Internal Working**.

---

# 2. 30-Second Interview Answer

> **"If two different keys have the same `hashCode()`, HashMap treats it as a collision. Both keys can map to the same bucket. HashMap does not consider them the same key just because their hash codes are equal. It compares the hash and then uses `equals()` to determine whether the keys are logically equal. If `equals()` returns false, both entries are stored in the same bucket. Depending on the bucket structure and collision level, they may be stored in a linked structure or a Red-Black tree in modern Java. If `equals()` returns true, they represent the same logical key and the existing value is replaced."**

---

# 3. First Important Point

### Same `hashCode()` does NOT mean same key.

This is the most important point.

```text
hashCode() same
      ↓
Collision
      ↓
Same bucket possible
      ↓
Check equals()
```

Not:

```text
same hashCode
      ↓
same key ❌
```

---

# 4. Simple Example

Suppose:

```java
class Employee {
    int id;

    Employee(int id) {
        this.id = id;
    }

    @Override
    public int hashCode() {
        return 100;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Employee other)) {
            return false;
        }
        return this.id == other.id;
    }
}
```

Now:

```java
Employee e1 = new Employee(101);
Employee e2 = new Employee(102);
```

Both return:

```text
hashCode(e1) = 100
hashCode(e2) = 100
```

But:

```text
e1.equals(e2) = false
```

Therefore they are different logical keys.

---

# 5. What Happens During `put()`?

```java
Map<Employee, String> map = new HashMap<>();

map.put(e1, "Nirbhay");
map.put(e2, "Rahul");
```

Flow:

```text
             e1
              ↓
          hashCode()
              ↓
             100
              ↓
          Bucket X
              ↓
       [e1 → Nirbhay]
```

Then:

```text
             e2
              ↓
          hashCode()
              ↓
             100
              ↓
          Bucket X
              ↓
       Collision detected
              ↓
          equals(e1,e2)?
              ↓
             false
              ↓
       Store second entry
```

Final concept:

```text
Bucket X
   ↓
[e1 → Nirbhay]
   ↓
[e2 → Rahul]
```

---

# 6. Why Does HashMap Use `equals()`?

`hashCode()` is used to narrow down where the entry should be searched.

`equals()` identifies whether two keys are logically the same.

Think:

```text
hashCode()
    ↓
Which bucket?

 equals()
    ↓
Which exact key?
```

So:

> **`hashCode()` helps locate the bucket; `equals()` helps identify the exact key.**

---

# 7. Two Cases You Must Remember

There are two different scenarios.

## Case 1 — Same hash, `equals()` false

```text
hashCode same
      ↓
equals false
      ↓
Different keys
      ↓
Collision
      ↓
Both entries can coexist
```

## Case 2 — Same hash, `equals()` true

```text
hashCode same
      ↓
equals true
      ↓
Same logical key
      ↓
Existing value replaced
```

---

# 8. Case 1 Example — Different Keys

```java
map.put(e1, "Nirbhay");
map.put(e2, "Rahul");
```

Assume:

```text
hashCode(e1) = 100
hashCode(e2) = 100
```

and:

```text
e1.equals(e2) = false
```

Then:

```text
Bucket
  ↓
e1 → Nirbhay
  ↓
e2 → Rahul
```

Both are stored.

---

# 9. Case 2 Example — Same Logical Key

Suppose:

```java
Employee e1 = new Employee(101);
Employee e2 = new Employee(101);
```

Both have:

```text
hashCode = 100
```

and:

```text
e1.equals(e2) = true
```

Then:

```java
map.put(e1, "Nirbhay");
map.put(e2, "Rahul");
```

Result:

```text
Employee(101) → Rahul
```

The second `put()` replaces the value for the same logical key.

---

# 10. Collision Is Normal

Collision itself is not a bug.

HashMap is designed to handle collisions.

Why are collisions unavoidable?

Because:

```text
Number of possible objects
        ↓
Potentially huge

hashCode range
        ↓
Finite integer range
```

Multiple objects can therefore produce the same hash value.

---

# 11. Collision vs Same Hash

Be precise in interview.

### Same hashCode

```text
key1.hashCode() == key2.hashCode()
```

This means the hash values are equal.

### Collision in HashMap

Two keys ultimately map to the same bucket.

Because bucket selection uses the processed hash and table capacity, even different hash codes can potentially map to the same bucket.

Therefore:

> **Same hashCode guarantees the same hash input to bucket selection, so with the same table state they map to the same bucket; but a collision can also happen when hash codes differ but bucket indices are equal.**

This is a strong interview-level distinction.

---

# 12. Bucket-Level Flow

```text
Key 1
  ↓
hashCode = 100
  ↓
hash spreading
  ↓
Bucket 5

Key 2
  ↓
hashCode = 100
  ↓
hash spreading
  ↓
Bucket 5
```

Now HashMap has to distinguish the entries.

```text
Bucket 5
   ↓
Node 1
   ↓
Node 2
```

It uses key equality checks.

---

# 13. What if `equals()` Is Not Overridden?

Suppose a custom class only overrides `hashCode()`:

```java
class Employee {
    int id;

    @Override
    public int hashCode() {
        return 100;
    }
}
```

If `equals()` is inherited from `Object`, then equality is normally based on object identity.

So:

```java
Employee e1 = new Employee();
Employee e2 = new Employee();
```

Even though:

```text
hashCode(e1) = 100
hashCode(e2) = 100
```

we can still have:

```text
e1.equals(e2) = false
```

Therefore both can be stored.

---

# 14. Why `hashCode()` Alone Is Not Enough

Imagine HashMap only used:

```java
hashCode()
```

Then:

```text
Same hash
   ↓
HashMap would think same key
```

That would be incorrect.

Example:

```text
Employee 101
Employee 102
```

can have the same hash.

Therefore HashMap needs `equals()` to distinguish them.

---

# 15. HashMap Lookup with Collision

Suppose:

```java
map.put(e1, "Nirbhay");
map.put(e2, "Rahul");
```

Then:

```java
map.get(e2);
```

Flow:

```text
get(e2)
   ↓
hashCode(e2)
   ↓
calculate bucket
   ↓
Bucket X
   ↓
compare candidate hash/key
   ↓
compare equals()
   ↓
e2 found
   ↓
return "Rahul"
```

---

# 16. Collision Chain

In a linked bucket:

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
```

If searching for Node C:

```text
A → not equal
 ↓
B → not equal
 ↓
C → equals true
 ↓
return value
```

This is why excessive collisions can degrade lookup performance.

---

# 17. Java 8+ Treeification

If a bucket becomes heavily collided, modern HashMap implementations can convert the bucket structure into a Red-Black tree, subject to implementation thresholds and table capacity conditions.

Conceptually:

```text
Linked structure

A
↓
B
↓
C
↓
D
↓
...

       ↓

Red-Black Tree
```

This improves lookup in the treeified bucket toward:

```text
O(log n)
```

instead of linear traversal:

```text
O(n)
```

---

# 18. Important Java 8 Interview Trap

Do NOT say:

> **"If 8 keys collide, HashMap always converts the bucket into a tree."**

That's incomplete.

Treeification depends on the implementation's thresholds and table capacity.

Common OpenJDK constants include:

```text
TREEIFY_THRESHOLD = 8
MIN_TREEIFY_CAPACITY = 64
```

If the table is too small, HashMap can resize instead of treeifying.

---

# 19. Worst-Case Complexity

### Before treeification

A heavily collided linked bucket can result in:

```text
O(n)
```

lookup.

### After treeification

Tree lookup is approximately:

```text
O(log n)
```

for that bucket.

### Average HashMap lookup

With good hash distribution:

```text
O(1)
```

So the interview answer should be:

> **"HashMap provides expected O(1) lookup with good hash distribution; collisions can degrade a bucket's lookup, with treeification improving heavy-collision behavior."**

---

# 20. Important `hashCode()` / `equals()` Contract

Java rule:

If:

```java
a.equals(b) == true
```

then:

```java
a.hashCode() == b.hashCode()
```

must be true.

But:

```text
same hashCode
```

does NOT require:

```text
equals true
```

This is the exact reason collisions are possible.

---

# 21. Visual Mental Model

Think of a HashMap like an apartment building.

```text
Hash
 ↓
Apartment number
 ↓
Bucket
```

Multiple people can be assigned to the same apartment number because the hash space and bucket space are limited.

Then:

```text
Bucket
 ↓
Check identity/equality
 ↓
Find exact person
```

So:

```text
Hash = Location hint
Equals = Exact identity check
```

---

# 22. Interviewer Follow-Up

### Interviewer:

> If two keys have the same hashCode, are they equal?

### Correct answer:

> **"No. Same hashCode only means their hash values are equal. They can still be different keys. HashMap uses equals() to determine logical key equality."**

---

# 23. Interviewer Follow-Up

### Interviewer:

> Can two different hashCodes result in the same bucket?

### Correct answer:

> **"Yes. Bucket selection depends on the hash and table capacity, so different hash values can map to the same bucket index. That is also a collision."**

---

# 24. Interviewer Follow-Up

### Interviewer:

> If `hashCode()` is same and `equals()` is false, what happens?

### Correct answer:

> **"Both keys can be stored in the same bucket because they are different logical keys. HashMap maintains the collision structure and uses equals() during lookup to identify the correct entry."**

---

# 25. Interviewer Follow-Up

### Interviewer:

> If `hashCode()` and `equals()` are both same, what happens?

### Correct answer:

> **"They represent the same logical key, so inserting the second mapping replaces the existing value rather than creating another mapping."**

---

# 26. Real-World Example

Suppose an Order Service stores:

```java
Map<OrderKey, Order> orders;
```

Two `OrderKey` objects might have the same hash:

```text
OrderKey(1001) → hash 500
OrderKey(2001) → hash 500
```

But:

```text
equals = false
```

Both orders must remain accessible.

HashMap handles this through collision handling.

---

# 27. Bad Hash Function Example

```java
@Override
public int hashCode() {
    return 1;
}
```

For every key:

```text
hash = 1
```

Therefore:

```text
Many keys
   ↓
Same bucket
   ↓
Heavy collisions
```

Modern HashMap can treeify a sufficiently large collided bucket, but a constant hash is still a poor design because it destroys normal hash distribution.

---

# 28. What Makes a Good `hashCode()`?

A good hash function should:

```text
1. Be consistent while key state is unchanged
2. Respect equals/hashCode contract
3. Distribute keys reasonably across buckets
4. Avoid unnecessary collisions
```

For custom classes, use relevant immutable fields.

Example:

```java
@Override
public int hashCode() {
    return Objects.hash(id, name);
}
```

---

# 29. Mutable Key Problem

Suppose:

```java
Employee e = new Employee(101);
map.put(e, "Nirbhay");
```

If `id` participates in `hashCode()` and then changes:

```text
id = 101
   ↓
Bucket 5

id changed to 202
   ↓
New hash
   ↓
Bucket 10
```

The object may physically remain in the old bucket while lookup uses the new hash.

Result:

```text
map.get(e)
   ↓
may not find it ❌
```

Therefore:

> **Use stable/immutable key fields in HashMap.**

---

# 30. Complete Flow to Memorize

```text
Two Keys
   ↓
Same hashCode?
   ↓
YES
   ↓
Same bucket
   ↓
Collision
   ↓
Compare keys
   ↓
 equals()?
   ├───────────────┐
   │               │
  YES             NO
   │               │
   ↓               ↓
Same logical     Different
key              logical keys
   │               │
   ↓               ↓
Replace value    Store both
                   │
                   ↓
            Linked / Tree structure
```

---

# 31. 2-Minute Interview Answer

> **"When two keys have the same hashCode in a HashMap, it is called a hash collision. Both keys can map to the same bucket. However, HashMap does not assume that the keys are equal. It compares the hash information and then uses equals() to determine logical key equality. If equals() returns false, both entries are stored in the same bucket using the collision structure. If equals() returns true, they are considered the same logical key and the new value replaces the old value. In modern Java, if a bucket becomes heavily collided and the implementation's treeification conditions are satisfied, the bucket can be converted into a Red-Black tree, improving lookup behavior from linear toward logarithmic. Also, a collision doesn't necessarily require identical hashCode values; different hashes can still map to the same bucket because bucket selection depends on table capacity."**

---

# 32. 30-Second Hinglish Answer

> **"Agar do keys ka same hashCode hai, to HashMap mein collision ho sakta hai aur dono same bucket mein aa sakti hain. Lekin same hashCode ka matlab same key nahi hota. HashMap `equals()` check karta hai. Agar `equals()` false hai to dono different keys hain aur same bucket mein store hongi. Agar `equals()` true hai to same logical key hai aur new value old value ko replace karegi. Heavy collision mein modern Java HashMap bucket ko Red-Black tree mein convert kar sakta hai, conditions ke according."**

---

# 33. Memory Trick

## **Same Hash → Same Bucket → Check Equals → Store or Replace**

```text
Same Hash
   ↓
Collision
   ↓
Equals?
 ┌───────┴───────┐
YES             NO
 ↓                ↓
Replace          Store both
```

One-line memory:

> **"Same hash is collision, equals decides identity."**

---

# 34. Common Interview Mistakes

### ❌ Mistake 1

> Same hashCode means same key.

### ✅ Correct

```text
Same hashCode ≠ same key
```

---

### ❌ Mistake 2

> HashMap uses only hashCode.

### ✅ Correct

```text
hashCode → bucket
 equals  → exact key
```

---

### ❌ Mistake 3

> Collision means HashMap loses one value.

### ✅ Correct

Different keys with same hash can coexist.

---

### ❌ Mistake 4

> Every collision immediately becomes a Red-Black tree.

### ✅ Correct

Treeification depends on collision count and table capacity/implementation thresholds.

---

### ❌ Mistake 5

> Different hashCodes can never collide.

### ✅ Correct

Different hashes can still map to the same bucket index.

---

# 35. Follow-Up Questions

- Why does HashMap need both `hashCode()` and `equals()`?
- What happens if `equals()` is overridden but `hashCode()` is not?
- What happens if `hashCode()` always returns the same value?
- How does Java 8 handle collisions?
- What is treeification?
- What is `TREEIFY_THRESHOLD`?
- Why is `MIN_TREEIFY_CAPACITY` important?
- What happens during HashMap resize?
- Why are mutable keys dangerous?
- What is the difference between collision and duplicate key?

---

## Status

✅ **Q31 Solution Completed**

Previous: [Q30 — HashMap Internal Working](../Q30-HashMap-Internal-Working/README.md)

Next: **Q32 — What is the time complexity before and after Java 8 collision handling?**
