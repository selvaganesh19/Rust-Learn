[← 5. Functions and control flow](../part-a-the-language-mapped-from-python/05-functions-and-control-flow.md) · [Index](../README.md) · [7. Structs and `impl` — the class replacement →](../part-a-the-language-mapped-from-python/07-structs-and-impl-the-class-replacement.md)

## 6. Collections — Vec, HashMap, HashSet, tuple

**In one line:** the same four tools as Python, but every collection holds exactly one element type and tells you clearly when something may be missing.

```rust
use std::collections::{BTreeMap, HashMap, HashSet, VecDeque};

fn main() {
    // ── Vec<T> = list ───────────────────────────────────────────────
    let mut v = vec![3, 7, 1, 9, 4];
    v.push(6);                            // append
    let last = v.pop();                   // Option<i32>: Some(6), or None if empty
    v.sort();                             // in place
    println!("{v:?} {last:?} len={} first={:?}", v.len(), v.first());
    println!("{:?} {:?} {}", &v[1..3], v.iter().max(), v.contains(&7));
    println!("{:?}", v.get(99));          // None. But v[99] would PANIC (like IndexError)

    // ── array [T; N] = fixed size, lives on the stack ───────────────
    let arr: [f32; 3] = [0.1, 0.2, 0.3];
    let total: f32 = arr.iter().sum();

    // ── tuple ───────────────────────────────────────────────────────
    let emp = ("Selva", 25, 9.1);
    let (name, age, _cgpa) = emp;         // unpacking
    println!("{name} {age} {} {total}", emp.0);

    // ── HashMap<K, V> = dict ────────────────────────────────────────
    let mut stock: HashMap<String, u32> = HashMap::new();
    stock.insert("idli".to_string(), 40);
    stock.insert("dosa".to_string(), 25);

    *stock.entry("dosa".to_string()).or_insert(0) += 5;      // d["dosa"] = d.get("dosa", 0) + 5
    println!("{:?} {:?}", stock.get("idli"), stock.get("vada"));    // Some(40) None
    println!("{}", stock.get("vada").copied().unwrap_or(0));        // d.get("vada", 0)

    if let Some(qty) = stock.get_mut("idli") {
        *qty -= 1;
    }
    stock.remove("dosa");
    for (item, qty) in &stock {
        println!("{item} -> {qty}");
    }

    // Frequency counter (your Counter from Q15)
    let mut freq: HashMap<char, usize> = HashMap::new();
    for ch in "programming".chars() {
        *freq.entry(ch).or_default() += 1;
    }
    let mut top: Vec<(char, usize)> = freq.into_iter().collect();
    top.sort_by(|a, b| b.1.cmp(&a.1).then(a.0.cmp(&b.0)));   // count desc, then letter asc
    println!("{:?}", &top[..3]);          // [('g', 2), ('m', 2), ('r', 2)]

    // ── HashSet<T> = set ────────────────────────────────────────────
    let a: HashSet<i32> = [1, 2, 3, 4].into();
    let b: HashSet<i32> = [3, 4, 5, 6].into();
    let mut both: Vec<i32> = a.intersection(&b).copied().collect();
    both.sort();
    println!("{both:?} {}", a.contains(&2));

    // ── VecDeque = deque, BTreeMap = dict that stays sorted by key ──
    let mut q = VecDeque::from([1, 2, 3]);
    q.push_front(0);
    q.pop_back();
    let mut sorted = BTreeMap::new();
    sorted.insert("b", 2);
    sorted.insert("a", 1);
    println!("{q:?} {sorted:?}");
}
```

**Python → Rust method map**

| Python | Rust |
|---|---|
| `lst.append(x)` | `v.push(x)` |
| `lst.extend(other)` | `v.extend(other)` |
| `lst.pop()` | `v.pop()` → `Option<T>` |
| `lst.insert(i, x)` / `del lst[i]` | `v.insert(i, x)` / `v.remove(i)` |
| `len(lst)` / `not lst` | `v.len()` / `v.is_empty()` |
| `lst[-1]` | `v.last()` → `Option<&T>` |
| `lst[1:3]` | `&v[1..3]` (a view, not a copy) |
| `x in lst` | `v.contains(&x)` |
| `sorted(lst)` | `let mut s = v.clone(); s.sort();` |
| `lst.sort(key=len)` | `v.sort_by_key(\|s\| s.len())` |
| `lst[::-1]` | `v.iter().rev()` or `v.reverse()` |
| `d[k]` | `d[&k]` (panics if missing) or `d.get(&k)` |
| `d.get(k, default)` | `d.get(&k).copied().unwrap_or(default)` |
| `d.setdefault(k, []).append(x)` | `d.entry(k).or_default().push(x)` |
| `k in d` | `d.contains_key(&k)` |
| `d.items()` | `for (k, v) in &d` |
| `a \| b`, `a & b`, `a - b` (sets) | `a.union(&b)`, `a.intersection(&b)`, `a.difference(&b)` |

**Inside the machine**

- `Vec<T>` is the same three words as `String`: pointer, length, capacity. When it is full it allocates a bigger buffer and moves everything (amortised O(1) push, like a Python list).
- `HashMap` is a "SwissTable" (the `hashbrown` design). The default hash function is SipHash with a random seed per program run — slower than simple hashes, but an attacker cannot craft keys that all collide. For hot loops with integer keys, people swap in a faster hasher such as `rustc-hash` or `ahash`.
- A Python `dict` keeps insertion order. `HashMap` does **not** — iteration order is random. Need order? `BTreeMap` (sorted by key) or the `indexmap` crate (insertion order).

**Trap — mixed types:** `[1, "two", 3.0]` is impossible in a `Vec`. If you really need different shapes in one list, make an `enum` (Section 8). That is the Rust answer to "list of anything".

**Trap — floats as keys:** `HashMap<f64, _>` and `HashSet<f64>` do not compile, because `NaN != NaN` breaks the rules of hashing. For the same reason `v.sort()` does not work on floats: use `v.sort_by(|a, b| a.total_cmp(b))`.

---

[← 5. Functions and control flow](../part-a-the-language-mapped-from-python/05-functions-and-control-flow.md) · [Index](../README.md) · [7. Structs and `impl` — the class replacement →](../part-a-the-language-mapped-from-python/07-structs-and-impl-the-class-replacement.md)
