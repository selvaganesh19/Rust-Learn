[← 13. Ownership](../part-b-the-core-that-makes-rust-rust/13-ownership.md) · [Index](../README.md) · [15. Lifetimes →](../part-b-the-core-that-makes-rust-rust/15-lifetimes.md)

## 14. Borrowing

**In one line:** a reference lets you use a value without owning it, under one rule — at any moment you may have **either** any number of readers (`&T`) **or** exactly one writer (`&mut T`), never both.

**Python you know:** every variable is a shared, mutable reference. That is why changing a list while looping over it gives strange results, and why two parts of a program can silently overwrite each other's data.

Think of a Google Doc with a strict rule: many people can open it in view mode together, or one person can open it in edit mode, but not at the same time. Nobody ever reads a half-edited document.

```rust
fn add_bonus(scores: &mut Vec<i32>, bonus: i32) {   // exclusive, may modify
    for s in scores.iter_mut() {
        *s += bonus;                                // `*` = go to the value behind the reference
    }
}

fn average(scores: &[i32]) -> f64 {                 // shared, read only
    scores.iter().sum::<i32>() as f64 / scores.len() as f64
}

fn main() {
    let mut scores = vec![70, 85, 90];

    let first = &scores[0];        // shared borrow starts
    // scores.push(100);           // error[E0502]: cannot borrow `scores` as mutable
    //                             //               because it is also borrowed as immutable
    println!("{first}");           // last use of `first` -> the borrow ends here
    scores.push(100);              // now allowed

    add_bonus(&mut scores, 5);     // you must write &mut at the call site: mutation is visible

    let r1 = &scores;
    let r2 = &scores;              // many readers together: fine
    println!("{r1:?} {} {:.1}", r2.len(), average(&scores));

    // Python bug: removing from a list while looping over it.
    // In Rust that loop does not compile. The tool for the job:
    scores.retain(|&s| s < 100);
    println!("{scores:?}");

    // Two mutable borrows at once: not allowed
    let a = &mut scores;
    // let b = &mut scores;        // error[E0499]: cannot borrow `scores` as mutable more than once
    a.push(1);
    println!("{scores:?}");
}
```

**Why the rule exists — the real reason, not "because Rust says so"**

Look at the commented line `scores.push(100)` while `first` is alive. `first` is a pointer into the Vec's heap buffer. `push` may find the buffer full, allocate a bigger one, copy everything, and free the old buffer. Now `first` points to freed memory. In C++ this is the famous "iterator invalidation" bug; it compiles and crashes later. Rust's rule makes it impossible: while a reader exists, nobody can write.

The same rule gives thread safety for free. A data race needs two threads touching the same memory with at least one writing. "Many readers OR one writer" forbids exactly that — so the borrow checker that protects your single-threaded code also protects your multi-threaded code (Section 21).

**Borrow lifetimes are short ("non-lexical lifetimes"):** a borrow ends at its *last use*, not at the closing brace. That is why `scores.push(100)` is legal right after `println!("{first}")`.

| Symbol | Name | Can read | Can write | How many at once |
|---|---|---|---|---|
| `T` | Owned value | Yes | If declared `mut` | One owner |
| `&T` | Shared reference | Yes | No | Many |
| `&mut T` | Exclusive reference | Yes | Yes | Exactly one, and no `&T` alive |

**Function parameter guide**

| You want to… | Parameter type | Call it with |
|---|---|---|
| Read a list | `&[T]` | `&v` |
| Read text | `&str` | `&s` or `"literal"` |
| Change a list in place | `&mut Vec<T>` | `&mut v` |
| Keep the data (store it in a struct, send to a thread) | `Vec<T>` / `String` | `v` (moves) |

**Trap — `&Vec<T>` and `&String` as parameters:** they work, but `&[T]` and `&str` are strictly better: they accept more inputs (arrays, slices, literals) and cost the same. `cargo clippy` will remind you.

**Say this in an interview:** "The borrow checker enforces aliasing XOR mutability: many shared references or one exclusive reference. That single rule prevents use-after-free, iterator invalidation and data races, all at compile time."

---

[← 13. Ownership](../part-b-the-core-that-makes-rust-rust/13-ownership.md) · [Index](../README.md) · [15. Lifetimes →](../part-b-the-core-that-makes-rust-rust/15-lifetimes.md)
