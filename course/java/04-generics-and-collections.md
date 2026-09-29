# Java 04: Generics and Collections

Almost every API method returns a collection. Interviews often ask how `HashMap` works and which collection to choose.

## 1. The collection hierarchy

```
Iterable
 └── Collection
      ├── List      ordered, allows duplicates     ArrayList, LinkedList
      ├── Set       no duplicates                  HashSet, LinkedHashSet, TreeSet
      └── Queue     FIFO or priority               ArrayDeque, PriorityQueue
Map (not a Collection)  key to value               HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap
```

| Need | Choose | Why |
|------|--------|-----|
| A list, general use | `ArrayList` | Fast index access, cache friendly |
| Unique items, no order | `HashSet` | O(1) add and contains |
| Unique items, insertion order | `LinkedHashSet` | O(1) plus stable order |
| Sorted items | `TreeSet` / `TreeMap` | O(log n), red-black tree |
| Key-value, general use | `HashMap` | O(1) average |
| Stack or queue | `ArrayDeque` | Faster than `Stack` and `LinkedList` |
| Shared between threads | `ConcurrentHashMap` | Thread-safe without a global lock |

`LinkedList` is almost never the right choice. Insertion in the middle is O(1) only after you find the node, which is O(n).

## 2. Mutable and unmodifiable collections

Kotlin separates `List` and `MutableList` in the type. Java does not. `List<T>` can be mutable or not, and you find out at runtime.

```java
List<String> a = new ArrayList<>(List.of("x"));  // mutable
List<String> b = List.of("x", "y");              // unmodifiable, Java 9+. Rejects null.
List<String> c = Arrays.asList("x", "y");        // fixed size: set works, add throws
List<String> d = Collections.unmodifiableList(a); // read-only view: changes to a are visible in d
List<String> e = List.copyOf(a);                 // unmodifiable copy
List<String> f = stream.toList();                // unmodifiable, Java 16+
```

## 3. How `HashMap` works

1. `put(key, value)` calls `key.hashCode()` and maps the hash to a bucket index in an array.
2. If the bucket is empty, the entry goes there.
3. If the bucket has entries (collision), Java compares keys with `equals`. The same key replaces the value. A different key is added to the bucket.
4. A bucket is a linked list. From Java 8, a bucket with more than 8 entries becomes a red-black tree, so the worst case is O(log n) instead of O(n).
5. When the size passes capacity times load factor (default 16 times 0.75 = 12), the array doubles and all entries are redistributed (rehash).

Consequences:

- A bad `hashCode` (for example, it always returns 1) makes every operation slow.
- A key that changes after `put` is lost.
- `HashMap` allows one `null` key. `ConcurrentHashMap` allows no `null` keys or values.
- `HashMap` iteration order is not defined. Do not rely on it.

## 4. Iteration and `ConcurrentModificationException`

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4));
for (Integer n : nums) {
    if (n % 2 == 0) nums.remove(n); // ConcurrentModificationException
}

nums.removeIf(n -> n % 2 == 0);    // correct
```

The for-each loop uses an iterator. The iterator detects that the list changed outside of it and fails fast.

## 5. `Comparable` and `Comparator`

```java
record Student(String name, int score) implements Comparable<Student> {
    public int compareTo(Student o) { return Integer.compare(score, o.score); } // natural order
}

students.sort(Comparator.comparing(Student::score).reversed()
                        .thenComparing(Student::name));
```

Do not write `return a.score - b.score`. It overflows for large values.

## 6. Generics basics

```java
public class Box<T> {
    private final T value;
    public Box(T value) { this.value = value; }
    public T get() { return value; }
}

public static <T extends Comparable<T>> T max(List<T> list) { /* ... */ return null; }
```

Generics give compile-time type checks and remove casts.

## 7. Type erasure

The compiler removes generic types after it checks them. At runtime, `List<String>` and `List<Integer>` are both `List`. Consequences:

- You cannot write `new T()` or `new T[10]`.
- You cannot write `if (x instanceof List<String>)`.
- You cannot overload `void f(List<String>)` and `void f(List<Integer>)`. They have the same erased signature.
- Frameworks such as Jackson need a trick to keep the type: `new TypeReference<List<User>>() {}`. Spring uses `ParameterizedTypeReference`.

Kotlin has `reified` type parameters for inline functions. Java has nothing like that.

## 8. Wildcards and PECS

Generics are invariant: `List<Integer>` is not a `List<Number>`, even though `Integer` is a `Number`.

```java
List<Integer> ints = new ArrayList<>();
List<Number> nums = ints; // compile error
```

Wildcards fix this. The PECS rule: Producer `extends`, Consumer `super`.

```java
// Reads Numbers from src (src produces): ? extends
double sum(List<? extends Number> src) {
    double s = 0;
    for (Number n : src) s += n.doubleValue();
    return s;
    // src.add(1) is a compile error: the real list can be List<Double>
}

// Writes Integers into dst (dst consumes): ? super
void fill(List<? super Integer> dst) {
    dst.add(1); // OK: the list holds Integer or a supertype
}
```

Kotlin equivalent: `out T` is `? extends T`, and `in T` is `? super T`.

`Collections.copy(List<? super T> dest, List<? extends T> src)` is the classic example.

## 9. Arrays and lists

Arrays are covariant and check types at runtime. Generics are invariant and check at compile time.

```java
Object[] arr = new String[1];
arr[0] = 1; // compiles, then throws ArrayStoreException
```

## Try it

1. Write a `Person` class without `hashCode`. Put 3 equal persons into a `HashSet` and print the size. Fix it.
2. Use a `Person` with a mutable `name` as a `HashMap` key, change the name, then call `get`. Explain the result.
3. Write `static <T> void copy(List<? super T> dst, List<? extends T> src)`.
4. Sort a list of `Student` by score descending, then by name.
