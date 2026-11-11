[← 10. Traits and generics](../part-a-the-language-mapped-from-python/10-traits-and-generics.md) · [Index](../README.md) · [12. Modules, crates, tests →](../part-a-the-language-mapped-from-python/12-modules-crates-tests.md)

## 11. Closures and iterators

**In one line:** iterator chains are Rust's comprehensions — lazy like generators, and after optimisation as fast as a hand-written loop.

**Python you know:** `[x*x for x in nums if x % 2 == 0]`, `sum(...)`, `any(...)`, `zip`, `enumerate`, generators with `yield`, `lambda`.

```rust
use std::collections::HashMap;

// A custom iterator = your generator with `yield` (Q4, fib_gen)
struct Fib {
    a: u64,
    b: u64,
}

impl Iterator for Fib {
    type Item = u64;
    fn next(&mut self) -> Option<u64> {        // return None to stop
        let out = self.a;
        self.a = self.b;
        self.b += out;
        Some(out)
    }
}

fn make_adder(n: i32) -> impl Fn(i32) -> i32 {   // a function returning a closure
    move |x| x + n                               // `move` = the closure owns `n`
}

fn apply_twice<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(f(x))
}

fn main() {
    let nums = vec![1, 2, 3, 4, 5, 6];

    // Comprehensions
    let squares: Vec<i32> = nums.iter().map(|x| x * x).collect();              // [x*x for x in nums]
    let evens: Vec<i32> = nums.iter().copied().filter(|x| x % 2 == 0).collect(); // [x for x in nums if x%2==0]
    println!("{squares:?} {evens:?}");

    // Reductions
    let total: i32 = nums.iter().sum();
    let product: i32 = nums.iter().product();
    let digits = nums.iter().fold(0, |acc, x| acc * 10 + x);     // functools.reduce
    println!("{total} {product} {digits}");

    // Searching — these stop early, like any()/all()/next()
    println!(
        "{} {} {:?} {:?}",
        nums.iter().any(|&x| x > 5),
        nums.iter().all(|&x| x > 0),
        nums.iter().find(|&&x| x > 3),         // first match
        nums.iter().position(|&x| x == 4),     // index of first match
    );

    // zip, enumerate, rev, skip, take, chain
    let names = ["asha", "ravi", "meena"];
    let marks = [91, 78, 85];
    let report: HashMap<&str, i32> = names.iter().copied().zip(marks).collect();   // dict(zip(...))
    let topper = names.iter().zip(marks).max_by_key(|(_, m)| *m);
    println!("{} {topper:?}", report["ravi"]);

    let middle: Vec<i32> = nums.iter().copied().skip(1).take(3).rev().collect();
    let joined: Vec<i32> = nums.iter().copied().chain([7, 8]).collect();
    println!("{middle:?} {joined:?}");

    // Flatten, windows, chunks, partition
    let nested = vec![vec![1, 2], vec![3], vec![4, 5]];
    let flat: Vec<i32> = nested.into_iter().flatten().collect();
    let diffs: Vec<i32> = nums.windows(2).map(|w| w[1] - w[0]).collect();
    let batches: Vec<&[i32]> = nums.chunks(4).collect();          // batching for inference!
    let (small, large): (Vec<i32>, Vec<i32>) = nums.iter().partition(|&&x| x < 4);
    println!("{flat:?} {diffs:?} {batches:?} {small:?} {large:?}");

    // Strings
    let caps: String = "hello world"
        .split(' ')
        .map(|w| {
            let mut c = w.chars();
            match c.next() {
                Some(first) => first.to_uppercase().collect::<String>() + c.as_str(),
                None => String::new(),
            }
        })
        .collect::<Vec<_>>()
        .join(" ");
    println!("{caps}");

    // Lazy + infinite: nothing runs until something pulls values
    let fibs: Vec<u64> = Fib { a: 0, b: 1 }.take_while(|&x| x < 100).filter(|x| x % 2 == 0).collect();
    println!("{fibs:?}");                      // [0, 2, 8, 34]

    // Closures capture their surroundings
    let threshold = 3;
    let above = |x: &&i32| **x > threshold;    // borrows `threshold`
    let mut seen = 0;
    let mut count_it = |_x: i32| seen += 1;    // mutably borrows `seen`  (FnMut)
    count_it(1);
    count_it(2);
    let add5 = make_adder(5);
    println!("{} {seen} {} {}", nums.iter().filter(above).count(), add5(10), apply_twice(&add5, 1));
}
```

**Three ways to iterate — this confuses everyone for a week**

| Call | You get | The collection after |
|---|---|---|
| `v.iter()` | `&T` (read) | Still usable |
| `v.iter_mut()` | `&mut T` (modify in place) | Still usable |
| `v.into_iter()` | `T` (owned) | Consumed, gone |
| `for x in &v` | Same as `v.iter()` | |
| `for x in &mut v` | Same as `v.iter_mut()` | |
| `for x in v` | Same as `v.into_iter()` | |

That is why you see `|&x|`, `.copied()` and `**x` in the code above: `iter()` hands out references, and `filter` passes a reference to that reference. For small `Copy` types, put `.copied()` right after `.iter()` and work with plain values.

**The three closure traits**

| Trait | The closure… | Can be called | Example |
|---|---|---|---|
| `Fn` | only reads what it captured | many times, even from many threads | `\|x\| x > threshold` |
| `FnMut` | changes what it captured | many times | `\|x\| seen += 1` |
| `FnOnce` | moves a captured value out | once | `move \|\| drop(data)` |

**Inside the machine:** `.map().filter().sum()` does not build temporary lists. Each adaptor is a tiny struct wrapping the previous one; `sum()` drives the chain by calling `next()`. After inlining, the compiler sees one simple loop and often auto-vectorises it with SIMD instructions. A closure is an anonymous struct holding its captured variables, with one method — no heap allocation unless you put it in a `Box`. In Python every lambda call is an interpreter-level function call.

**Trap — forgetting to consume:** `nums.iter().map(|x| x * 2);` does nothing, and the compiler warns "iterators are lazy". You need a consumer: `collect()`, `sum()`, `count()`, `for_each()`, or a `for` loop.

**Trap — `collect()` needs to know the target:** write `let v: Vec<_> = ...collect();` or `.collect::<Vec<_>>()`. The same `collect()` can build a `Vec`, `String`, `HashMap`, `HashSet`, even `Result<Vec<T>, E>` (stops at the first error) — the type you ask for decides.

**Say this in an interview:** "Iterators are lazy and zero-cost: adaptors are structs that get inlined into a single loop, with bounds checks removed. `iter` borrows, `iter_mut` borrows mutably, `into_iter` consumes."

---

[← 10. Traits and generics](../part-a-the-language-mapped-from-python/10-traits-and-generics.md) · [Index](../README.md) · [12. Modules, crates, tests →](../part-a-the-language-mapped-from-python/12-modules-crates-tests.md)
