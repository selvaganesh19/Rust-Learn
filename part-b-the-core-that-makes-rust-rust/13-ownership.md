[← 12. Modules, crates, tests](../part-a-the-language-mapped-from-python/12-modules-crates-tests.md) · [Index](../README.md) · [14. Borrowing →](../part-b-the-core-that-makes-rust-rust/14-borrowing.md)

## 13. Ownership

**In one line:** every value has exactly one owner; when the owner goes out of scope the value is freed; giving the value to someone else *moves* it, and the old name becomes unusable.

**Python you know:** `b = a` makes two names for the same object. A reference count tracks how many names exist, and the garbage collector frees the object when the count reaches zero (your Q39).

Think of the RC book of a bike. Only one name is on it. If you sell the bike, the RC goes to the buyer and you cannot ride it any more. You can *lend* the bike to a friend (that is borrowing, next section), but the RC stays with you. Rust tracks the RC book for every value at compile time.

**The three rules**

1. Each value has one owner (a variable).
2. When the owner goes out of scope, the value is dropped (memory freed, file closed).
3. Assigning or passing a value moves ownership — unless the type is `Copy`.

```rust
fn consume(v: Vec<i32>) -> i32 {      // takes ownership
    v.iter().sum()
}                                     // `v` is dropped here: heap memory freed

fn inspect(v: &[i32]) -> i32 {        // only borrows
    v.iter().sum()
}

fn build() -> Vec<i32> {              // returns ownership to the caller
    vec![7, 8, 9]
}

fn main() {
    let a = String::from("model.bin");
    let b = a;                        // MOVE. The 24-byte header is copied, the heap text is not.
    // println!("{a}");               // error[E0382]: borrow of moved value: `a`

    let c = b.clone();                // deep copy: new heap allocation, both alive
    println!("{b} {c}");

    let x = 5;
    let y = x;                        // i32 is `Copy`: bits are duplicated, both alive
    println!("{x} {y}");

    let v = vec![1, 2, 3];
    let total = consume(v);           // v moved into the function
    // println!("{}", v.len());       // error[E0382]: borrow of moved value: `v`

    let v2 = build();                 // ownership comes out of a function
    let total2 = inspect(&v2);        // lend it; we still own it
    println!("{total} {total2} {v2:?}");

    {
        let temp = String::from("scratch");
        println!("{temp}");
    }                                 // `temp` freed exactly here — no GC, no waiting
}
```

**Inside the machine**

```
let a = String::from("hi");          let b = a;

 a ┌─────────┐     HEAP               a ┌─────────┐  (compiler forbids using it)
   │ ptr   ──┼──► ┌───┬───┐             │ ░░░░░░░ │
   │ len 2   │    │ h │ i │             └─────────┘
   │ cap 2   │    └───┴───┘           b ┌─────────┐     HEAP
   └─────────┘                          │ ptr   ──┼──► ┌───┬───┐
                                        │ len 2   │    │ h │ i │
                                        │ cap 2   │    └───┴───┘
                                        └─────────┘
```

A move is a plain copy of the few stack bytes. The heap is not touched. Nothing happens at runtime to "invalidate" `a` — the compiler simply refuses to compile code that uses it. Because exactly one owner exists, the compiler knows the one place where the memory must be freed and inserts that call for you. So Rust has no garbage collector, no reference counts, and still no manual `free()`.

**`Copy` types** (assignment duplicates, old name stays valid): all integers and floats, `bool`, `char`, shared references `&T`, and tuples/arrays made only of `Copy` types.
**Move types:** `String`, `Vec`, `HashMap`, `Box`, anything that owns heap memory, and any struct that contains one of them.

| | Python | Rust |
|---|---|---|
| `b = a` for a list | Two names, one object, refcount = 2 | `a` is moved; only `b` is usable |
| Who frees memory | Refcount + cyclic GC, at runtime | Compiler-inserted `drop` at end of scope |
| Cost of tracking | An increment/decrement on every assignment | Zero at runtime |
| Passing to a function | Callee can secretly mutate your list (your Q1 `clear_cart` bug) | The signature says it: `Vec<T>` takes, `&[T]` reads, `&mut Vec<T>` changes |

**Trap — "I'll just `.clone()` everything":** it makes the error go away and the program correct, but each clone of a `Vec` or `String` is a heap allocation plus a full copy. While learning, cloning is fine. In a hot loop it is the first thing to remove — borrow instead.

**Say this in an interview:** "Ownership gives each value a single owner, so the compiler knows statically where to free it. Assignment of non-`Copy` types is a move — a shallow bitwise copy plus the source being made unusable at compile time. That is how Rust gets memory safety without a garbage collector."

---

[← 12. Modules, crates, tests](../part-a-the-language-mapped-from-python/12-modules-crates-tests.md) · [Index](../README.md) · [14. Borrowing →](../part-b-the-core-that-makes-rust-rust/14-borrowing.md)
