# Part G — Practice and standing out

[← Back to index](../README.md)

**In this module**

- [34. Your Python problems, solved in Rust](#34-your-python-problems-solved-in-rust)
- [35. Compiler errors decoded](#35-compiler-errors-decoded)
- [36. Interview Q&A](#36-interview-qa)
- [37. Projects and an 8-week plan](#37-projects-and-an-8-week-plan)
- [38. One-page cheat sheet](#38-one-page-cheat-sheet)

---

## 34. Your Python problems, solved in Rust

The same problems from your `PYTHON_BASIC_CODING.md`. Every function is checked by an `assert` in `main`, so if this file runs silently to the last line, every answer is right.

```rust
use std::collections::{HashMap, HashSet};
use std::hash::Hash;

// Q1 — palindrome number (maths, no string)
fn is_palindrome_num(n: i64) -> bool {
    if n < 0 {
        return false;
    }
    let (mut x, mut rev) = (n, 0);
    while x > 0 {
        rev = rev * 10 + x % 10;
        x /= 10;
    }
    rev == n
}

// Q2 — Armstrong number
fn is_armstrong(n: u32) -> bool {
    let digits = n.to_string();
    let power = digits.len() as u32;
    digits.chars().map(|c| c.to_digit(10).unwrap().pow(power)).sum::<u32>() == n
}

// Q3 — prime. `all` on an empty range is true, so 2 and 3 work.
fn is_prime(n: u64) -> bool {
    n >= 2 && (2..).take_while(|i| i * i <= n).all(|i| n % i != 0)
}

// Q4 — Fibonacci
fn fibonacci(n: usize) -> Vec<u64> {
    let mut out = Vec::with_capacity(n);
    let (mut a, mut b) = (0u64, 1u64);
    for _ in 0..n {
        out.push(a);
        (a, b) = (b, a + b);      // tuple assignment, like Python
    }
    out
}

// Q5 — factorial. Python ints never overflow; here overflow becomes None.
fn factorial(n: u32) -> Option<u64> {
    (1..=u64::from(n)).try_fold(1u64, |acc, x| acc.checked_mul(x))
}

// Q8 — sum of digits
fn digit_sum(n: i64) -> u32 {
    n.unsigned_abs().to_string().chars().filter_map(|c| c.to_digit(10)).sum()
}

// Q12 — palindrome string (ignore case, spaces, punctuation)
fn is_palindrome_str(s: &str) -> bool {
    let clean: Vec<char> = s.chars().filter(|c| c.is_alphanumeric()).flat_map(char::to_lowercase).collect();
    clean.iter().eq(clean.iter().rev())
}

// Q13 — vowels and consonants
fn count_vowels_consonants(s: &str) -> (usize, usize) {
    let (mut vowels, mut consonants) = (0, 0);
    for ch in s.to_lowercase().chars() {
        if "aeiou".contains(ch) {
            vowels += 1;
        } else if ch.is_alphabetic() {
            consonants += 1;
        }
    }
    (vowels, consonants)
}

// Q14 — reverse word order
fn reverse_words(s: &str) -> String {
    s.split_whitespace().rev().collect::<Vec<_>>().join(" ")
}

// Q15 — character frequency
fn char_freq(s: &str) -> HashMap<char, usize> {
    let mut freq = HashMap::new();
    for ch in s.chars() {
        *freq.entry(ch).or_insert(0) += 1;
    }
    freq
}

// Q16 / Q23 — remove duplicates, keep order. Generic: works for chars, ints, strings.
fn remove_duplicates<T: Hash + Eq + Clone>(items: &[T]) -> Vec<T> {
    let mut seen = HashSet::new();
    items.iter().filter(|x| seen.insert((*x).clone())).cloned().collect()
    // HashSet::insert returns false if the value was already there
}

// Q17 — anagram
fn is_anagram(a: &str, b: &str) -> bool {
    let key = |s: &str| {
        let mut chars: Vec<char> = s.to_lowercase().chars().collect();
        chars.sort_unstable();
        chars
    };
    key(a) == key(b)
}

// Q22 — second largest distinct value, one pass. None < Some(x) for every x.
fn second_largest(nums: &[i32]) -> Option<i32> {
    let (mut first, mut second) = (None, None);
    for &n in nums {
        let n = Some(n);
        if n > first {
            second = first;
            first = n;
        } else if n != first && n > second {
            second = n;
        }
    }
    second
}

// Q34 — group by
fn group_by_dept<'a>(employees: &[(&'a str, &'a str)]) -> HashMap<&'a str, Vec<&'a str>> {
    let mut groups: HashMap<&str, Vec<&str>> = HashMap::new();
    for &(name, dept) in employees {
        groups.entry(dept).or_default().push(name);
    }
    groups
}

// Q43 — FizzBuzz with a tuple match
fn fizzbuzz(n: u32) -> String {
    match (n % 3, n % 5) {
        (0, 0) => "FizzBuzz".to_string(),
        (0, _) => "Fizz".to_string(),
        (_, 0) => "Buzz".to_string(),
        _ => n.to_string(),
    }
}

// Q45 — missing number in 1..=n
fn missing_number(nums: &[u64], n: u64) -> u64 {
    n * (n + 1) / 2 - nums.iter().sum::<u64>()
}

// Q50 — pairs that sum to a target, O(n) with a set
fn find_pairs(nums: &[i32], target: i32) -> Vec<(i32, i32)> {
    let mut seen = HashSet::new();
    let mut pairs = Vec::new();
    for &n in nums {
        if seen.contains(&(target - n)) {
            pairs.push((target - n, n));
        }
        seen.insert(n);
    }
    pairs
}

// Q52 — transpose
fn transpose(m: &[Vec<i32>]) -> Vec<Vec<i32>> {
    if m.is_empty() {
        return Vec::new();
    }
    (0..m[0].len()).map(|c| m.iter().map(|row| row[c]).collect()).collect()
}

// Q54 — sort records by salary descending, then name ascending
#[derive(Debug, Clone, PartialEq)]
struct Employee {
    name: &'static str,
    salary: u32,
}

fn main() {
    assert!(is_palindrome_num(1221) && !is_palindrome_num(123) && !is_palindrome_num(-121));
    assert!(is_armstrong(153) && is_armstrong(371) && !is_armstrong(100));
    assert_eq!((1..50).filter(|&n| is_prime(n)).collect::<Vec<_>>(), [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]);
    assert_eq!(fibonacci(10), [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]);
    assert_eq!(factorial(10), Some(3_628_800));
    assert_eq!(factorial(25), None);                      // 25! does not fit in u64
    assert_eq!(digit_sum(-567), 18);
    assert!(is_palindrome_str("A man, a plan, a canal: Panama") && !is_palindrome_str("hello"));
    assert_eq!(count_vowels_consonants("Hello World"), (3, 7));
    assert_eq!(reverse_words("Hello World Python"), "Python World Hello");
    assert_eq!(char_freq("programming")[&'g'], 2);

    let letters: Vec<char> = "programming".chars().collect();
    assert_eq!(remove_duplicates(&letters).iter().collect::<String>(), "progamin");
    assert_eq!(remove_duplicates(&[1, 2, 2, 3, 4, 4, 5, 1]), [1, 2, 3, 4, 5]);

    assert!(is_anagram("Triangle", "Integral") && !is_anagram("hello", "world"));
    assert_eq!(second_largest(&[3, 7, 1, 9, 4, 6, 9, 2]), Some(7));
    assert_eq!(second_largest(&[5, 5, 5]), None);

    let groups = group_by_dept(&[("Asha", "Eng"), ("Ravi", "HR"), ("Meena", "Eng")]);
    assert_eq!(groups["Eng"], ["Asha", "Meena"]);

    assert_eq!((1..=15).map(fizzbuzz).last().unwrap(), "FizzBuzz");
    assert_eq!(missing_number(&[1, 2, 4, 5, 6], 6), 3);
    assert_eq!(find_pairs(&[1, 5, 3, 7, 2, 8, 4], 9), [(7, 2), (1, 8), (5, 4)]);
    assert_eq!(transpose(&[vec![1, 2, 3], vec![4, 5, 6]]), [[1, 4], [2, 5], [3, 6]]);

    let mut staff = vec![
        Employee { name: "Charlie", salary: 80_000 },
        Employee { name: "Alice", salary: 95_000 },
        Employee { name: "Bob", salary: 80_000 },
    ];
    staff.sort_by(|a, b| b.salary.cmp(&a.salary).then_with(|| a.name.cmp(b.name)));
    assert_eq!(staff.iter().map(|e| e.name).collect::<Vec<_>>(), ["Alice", "Bob", "Charlie"]);

    println!("all checks passed");
}
```

**What changed in your thinking, problem by problem**

| Python habit | Rust version | Lesson |
|---|---|---|
| `factorial(25)` just works | Returns `None` | Fixed-size integers: decide what overflow means |
| `second_largest` returns `None` or a number | `Option<i32>` in the signature | The caller is forced to handle "no answer" |
| `float('-inf')` as "nothing yet" | `None`, and `None < Some(x)` | No magic sentinel values |
| `Counter(s)` | `HashMap` + `entry().or_insert(0)` | One hash lookup instead of two |
| `s[::-1]` | `.chars().rev()` | Strings are UTF-8 bytes, characters need an iterator |
| `sorted(emps, key=lambda e: (-e.salary, e.name))` | `sort_by` + `cmp().then_with()` | Comparators compose, no negation trick |
| `x not in seen` then `seen.add(x)` | `seen.insert(x)` returns a bool | One operation, one lookup |

**Practice method that works:** take one problem from your Python file each day, write it in Rust *without* looking here, make the compiler happy, then compare. Twenty problems this way and the borrow checker stops feeling like an enemy.

---

## 35. Compiler errors decoded

Rust errors have a code. `rustc --explain E0382` prints a full explanation with examples. These are the ones you will meet in the first month.

| Code | Message (short) | What it really means | Usual fix |
|---|---|---|---|
| E0382 | use of moved value | You gave the value away and then used the old name | Pass `&x` instead, or `.clone()`, or restructure so the move is last |
| E0499 | cannot borrow as mutable more than once | Two `&mut` to the same thing are alive together | Finish using the first before creating the second; split into smaller scopes |
| E0502 | cannot borrow as mutable because also borrowed as immutable | A reader is still alive while you try to write | Use the reader's result first (copy the value out), then mutate |
| E0505 | cannot move out because it is borrowed | You tried to give away something a reference still points to | Drop or stop using the reference first |
| E0506 | cannot assign because it is borrowed | Same as above, for assignment | Same |
| E0507 | cannot move out of borrowed content / behind a reference | You have `&T` but tried to take the `T` out | `.clone()`, or `std::mem::take`, or `Option::take()`, or match on a reference |
| E0597 | does not live long enough | A reference outlives the value it points to | Move the owner to an outer scope, or return an owned value |
| E0106 | missing lifetime specifier | A returned or stored reference with no way to infer its source | Add `<'a>`, or return an owned type |
| E0373 | closure may outlive the current function | A thread or task closure borrows a local | Add `move` before the closure |
| E0277 | trait bound is not satisfied | The type lacks a capability (`Display`, `Send`, `Clone`, `Serialize`…) | Add `#[derive(...)]`, implement the trait, or add a bound `T: Trait` |
| E0308 | mismatched types | Expected one type, found another | Read "expected … found …". Often a stray `;`, a missing `&`, or `Some(...)` / `Ok(...)` forgotten |
| E0599 | no method named … found | The method does not exist for this type, or its trait is not imported | `use` the trait (e.g. `use rayon::prelude::*`), or check the type is what you think |
| E0282 | type annotations needed | `collect()` / `parse()` / `into()` cannot guess the target type | `let v: Vec<_> = ...` or `.collect::<Vec<_>>()` |
| E0384 | cannot assign twice to immutable variable | Missing `mut` | `let mut x` |
| E0596 | cannot borrow as mutable | The variable or reference is not `mut` | `let mut x`, or take `&mut` in the signature |
| E0425 / E0433 | cannot find value / failed to resolve | Name not in scope | Add a `use`, check spelling, check `mod` declaration, check `Cargo.toml` |
| E0616 | field is private | Used a private field from another module | Add `pub`, or use a method |
| E0004 | non-exhaustive patterns | A `match` misses a case | Add the missing arm |

**How to read a Rust error** (they look long; the structure is always the same):

```
error[E0502]: cannot borrow `scores` as mutable because it is also borrowed as immutable
 --> src/main.rs:5:5
  |
4 |     let first = &scores[0];
  |                  ------ immutable borrow occurs here          <- 1. where the first borrow started
5 |     scores.push(100);
  |     ^^^^^^^^^^^^^^^^ mutable borrow occurs here               <- 2. the line that conflicts
6 |     println!("{first}");
  |               ------- immutable borrow later used here        <- 3. why the first borrow is still alive
```

Three arrows, three facts. Fix any one of them and the error goes: stop using `first` later, or push before borrowing, or copy the value out (`let first = scores[0];`). Read the `help:` line at the bottom too — the compiler often prints the exact fix.

---

## 36. Interview Q&A

Short answers you can say aloud. The section number tells you where the full explanation is.

**Language core**

**Q1. How is Rust memory-safe without a garbage collector?** (13, 14)
Ownership: each value has one owner and is freed when the owner goes out of scope. Borrowing: references are checked at compile time so they can never outlive the value, and you may have many readers or one writer, never both. All checks happen at compile time, so there is no runtime cost.

**Q2. Move vs Copy vs Clone?** (13)
Move: bitwise copy of the stack part, and the source becomes unusable. Copy: the same bitwise copy, but the source stays valid — only for simple types with no heap data. Clone: an explicit, possibly expensive deep copy through `.clone()`.

**Q3. `String` vs `&str`?** (4)
`String` is an owned, growable, heap-allocated UTF-8 buffer: pointer, length, capacity. `&str` is a borrowed view: pointer and length. Take `&str` in parameters, return `String`.

**Q4. What are lifetimes?** (15)
Compile-time labels for how long references are valid. They do not change runtime behaviour. Mostly inferred; I annotate when a returned reference could come from more than one input or when a struct stores a reference.

**Q5. `Box` vs `Rc` vs `Arc`?** (16)
`Box`: single owner on the heap. `Rc`: shared ownership with a non-atomic count, single thread only. `Arc`: the same with an atomic count, safe across threads.

**Q6. What is interior mutability?** (16)
Mutating through a shared reference, with the borrow rule checked at runtime instead of compile time: `Cell`/`RefCell` for one thread, `Mutex`/`RwLock`/atomics for many.

**Q7. `Option`/`Result` vs exceptions? When do you panic?** (9)
Failure is part of the return type, so callers must handle it; `?` propagates. No hidden control flow. Panic only for bugs — broken invariants — never for expected failures like bad input or a missing file.

**Q8. Trait objects vs generics?** (10, 19)
Generics are monomorphised: static dispatch, inlinable, bigger binary. `dyn Trait` is a fat pointer with a vtable: dynamic dispatch, supports mixed types in one collection, no inlining.

**Q9. What is a zero-cost abstraction? Give an example.** (11, 19)
An abstraction that compiles to the same machine code as the hand-written low-level version. Iterator chains become one loop; `Option<&T>` is the size of a pointer; generics have no dispatch cost.

**Q10. `iter()` vs `into_iter()` vs `iter_mut()`?** (11)
`iter` yields `&T`, `iter_mut` yields `&mut T`, `into_iter` yields `T` and consumes the collection.

**Q11. Can a safe Rust program leak memory?** (16)
Yes. `Rc` cycles, `std::mem::forget`, `Box::leak`. Leaks are considered safe — they are not undefined behaviour. Rust guarantees no use-after-free and no data races, not "no leaks".

**Q12. What does `unsafe` allow, and what does it not?** (23)
Five things: dereference raw pointers, call unsafe functions, access mutable statics, implement unsafe traits, access union fields. It does not turn off the borrow checker or type checker.

**Internals**

**Q13. Walk me through compilation.** (17)
Source → AST → HIR (type check) → MIR (borrow check) → monomorphisation → LLVM IR → optimised machine code → linker. Safety checks are finished before LLVM ever sees the code.

**Q14. Why is `Vec<Vec<f32>>` a poor choice for an embedding matrix?** (18)
Each inner `Vec` is its own heap allocation, so rows are scattered: pointer chasing, cache misses, no SIMD across rows. One flat `Vec<f32>` with a known `dim` is contiguous and is what every tensor library does.

**Q15. How does Rust prevent data races?** (21)
`Send` and `Sync` marker traits plus the borrow rule. To share across threads a type must be `Sync`; to mutate shared data you must go through `Mutex`, `RwLock` or atomics. The compiler rejects anything else.

**Q16. How does async work underneath?** (22)
`async fn` compiles to a state machine implementing `Future`. An executor polls it; at each `.await` that is not ready it returns `Pending` and registers a waker. tokio multiplexes many such tasks over a small thread pool using the OS event queue.

**Q17. Why must inference not run directly in an async handler?** (22, 32)
It is CPU-bound and never yields, so it blocks a runtime worker thread and stalls every other task on it. Use `spawn_blocking` or a dedicated thread pool.

**Python + Rust**

**Q18. How do you call Rust from Python?** (26)
PyO3 for bindings, maturin to build wheels. `#[pyfunction]`, `#[pyclass]`, `#[pymodule]` generate the CPython glue.

**Q19. How do you pass a NumPy array without copying?** (26, 27)
Take `PyReadonlyArray<f32, D>` from the `numpy` crate. It borrows NumPy's buffer; `as_array()` gives an ndarray view, `as_slice()` a `&[f32]` if contiguous.

**Q20. What about the GIL?** (27)
A PyO3 function holds the GIL by default. For long work that does not touch Python objects I wrap it in `py.detach` (called `allow_threads` before PyO3 0.26), which releases the GIL so other Python threads run and my Rust code can use all cores with rayon.

**Q21. What happens if Rust panics inside a Python call?** (27)
PyO3 catches the unwind at the boundary and raises `PanicException` in Python. The interpreter survives. That is also why I do not set `panic = "abort"` for extension modules.

**Q22. When would you not use Rust?** (28)
When the hot path is already a vectorised native call, for training and exploration, for I/O-bound glue, and when nobody else on the team can maintain it. I profile before deciding.

**Q23. Rust vs C++ for ML infrastructure?**
Same performance class and the same LLVM backend. Rust adds compile-time memory and thread safety, a single build tool, and easy Python packaging with maturin. C++ has the larger existing ecosystem — CUDA kernels, most framework internals — so the two often sit side by side through FFI.

---

## 37. Projects and an 8-week plan

**How to actually stand out:** anyone can list "Rust" on a résumé. Very few can show a repository where a Python benchmark and a Rust-backed version of the same function sit side by side with numbers, tests, and a published wheel. Build these, in this order.

| # | Project | What it proves | Crates |
|---|---|---|---|
| 1 | `fastcount` — word/token frequency for big text files, as a Python package | PyO3, maturin, benchmarking against `collections.Counter` | pyo3, rayon |
| 2 | `rustcore` from Section 26, extended: save/load the index, batch search, a `pytest` suite | NumPy zero-copy, GIL release, flat memory layout | pyo3, numpy, rayon |
| 3 | A BPE tokenizer from scratch (train merges, encode, decode), with Python bindings | Algorithms, `&str` slicing, HashMap performance | pyo3, rayon |
| 4 | Embedding server: axum + a real model through candle or ort, `/embed` and `/search`, Dockerfile, load test | Async, state, `spawn_blocking`, deployment | axum, tokio, candle or ort, tokenizers |
| 5 | Mini RAG API: ingest files, chunk, embed, store in Qdrant or LanceDB, stream answers over SSE | The whole stack in Section 30.7 | axum, reqwest, qdrant-client |

For each one, the README must contain: the problem, a benchmark table (machine, data size, Python time, Rust time), how to install, and what you would do next. That README is what an interviewer reads.

**8-week plan** (about one hour on weekdays, more on weekends)

| Week | Read here | Do |
|---|---|---|
| 1 | Sections 1–6 | Install, rust-analyzer. Solve 10 problems from your Python file in Rust. |
| 2 | Sections 7–9 | Rustlings exercises up to "error handling". Rewrite 3 of your Python classes as struct + impl. |
| 3 | Sections 10–12 | Finish Rustlings. Write a small CLI (word count over a folder) with `clap`. |
| 4 | Sections 13–16 | Re-read ownership slowly. For every compiler error, look up its code in Section 35 before fixing it. |
| 5 | Sections 17–21 | Run the layout program. Parallelise your CLI with rayon and measure. Project 1. |
| 6 | Sections 22–28 | Build `rustcore` by typing it, not pasting. Project 2. |
| 7 | Sections 29–30 | candle examples: run a small BERT embedding model. Start Project 3 or 4. |
| 8 | Sections 31–33, 36 | Finish the server, write the READMEs, practise Section 36 aloud. |

**Free resources worth your time**

| Resource | Use it for |
|---|---|
| *The Rust Programming Language* ("the Book") — doc.rust-lang.org/book | The standard text. Read chapters 4, 10, 15, 16 twice. |
| Rustlings — github.com/rust-lang/rustlings | Small fix-the-code exercises in your terminal |
| Rust by Example — doc.rust-lang.org/rust-by-example | Looking up "how do I write X" |
| The PyO3 user guide — pyo3.rs | Everything in Part D, in depth |
| The Tokio tutorial — tokio.rs | Async, properly |
| candle repository, `candle-examples` folder | Real model inference code to read |
| The Burn Book — burn.dev | Training in Rust |
| Compiler Explorer — godbolt.org | Seeing what your code compiles to |
| docs.rs | API docs for every crate and version |

---

## 38. One-page cheat sheet

```rust
// ── Variables ────────────────────────────────────────────────────────
let x = 5;                      // immutable
let mut y = 5;                  // mutable
const N: usize = 10;            // constant
let x = x + 1;                  // shadowing

// ── Types ────────────────────────────────────────────────────────────
// i8..i128  u8..u128  isize usize  f32 f64  bool  char  ()  &str String
// [T; N] array   &[T] slice   (A, B) tuple   Vec<T>   HashMap<K, V>   HashSet<T>
// Option<T> = Some(T) | None           Result<T, E> = Ok(T) | Err(E)

// ── Ownership ────────────────────────────────────────────────────────
let a = String::from("x");
let b = a;                      // move: `a` is gone
let c = b.clone();              // deep copy
let r = &c;                     // shared borrow   (many allowed)
let m = &mut y;                 // exclusive borrow (only one, no readers)

// ── Functions ────────────────────────────────────────────────────────
fn read(v: &[i32]) -> i32 { v.iter().sum() }          // borrow
fn change(v: &mut Vec<i32>) { v.push(1); }            // mutable borrow
fn take(v: Vec<i32>) -> usize { v.len() }             // takes ownership
fn pick<'a>(a: &'a str, b: &'a str) -> &'a str { a }  // lifetime

// ── Struct, enum, trait ──────────────────────────────────────────────
#[derive(Debug, Clone, PartialEq)]
struct Point { x: f32, y: f32 }
impl Point {
    fn new(x: f32, y: f32) -> Self { Self { x, y } }
    fn len(&self) -> f32 { (self.x * self.x + self.y * self.y).sqrt() }
}
enum Shape { Circle(f32), Rect { w: f32, h: f32 } }
trait Area { fn area(&self) -> f32; }
impl Area for Shape {
    fn area(&self) -> f32 {
        match self {
            Shape::Circle(r) => 3.14159 * r * r,
            Shape::Rect { w, h } => w * h,
        }
    }
}

// ── Control flow ─────────────────────────────────────────────────────
if let Some(v) = maybe { }                    // one pattern
let Some(v) = maybe else { return; };         // or leave
for (i, item) in list.iter().enumerate() { }
while let Some(top) = stack.pop() { }
let n = loop { break 42; };

// ── Errors ───────────────────────────────────────────────────────────
let n: i32 = text.parse()?;                           // propagate
let n = text.parse::<i32>().unwrap_or(0);             // default
let v = map.get("k").ok_or("missing key")?;           // Option -> Result
let r = thing().map_err(|e| format!("failed: {e}"))?; // convert error

// ── Iterators ────────────────────────────────────────────────────────
v.iter().map(|x| x * 2).filter(|x| *x > 3).collect::<Vec<_>>();
v.iter().sum::<i32>();   v.iter().max();   v.iter().any(|&x| x > 0);
v.iter().zip(&w);        v.chunks(8);      v.windows(2);
v.sort();   v.sort_by(|a, b| b.cmp(a));   v.sort_by_key(|p| p.age);   v.dedup();   v.retain(|x| *x > 0);

// ── Smart pointers and threads ───────────────────────────────────────
Box::new(x)                 // heap, one owner
Rc::new(x)                  // shared, one thread
Arc::new(Mutex::new(x))     // shared + mutable, many threads
thread::spawn(move || { })  // new OS thread
data.par_iter().map(f)      // rayon: all cores

// ── Async ────────────────────────────────────────────────────────────
#[tokio::main] async fn main() { let (a, b) = tokio::join!(f(), g()); }
tokio::spawn(async move { });                 // background task
tokio::task::spawn_blocking(|| heavy());      // CPU work off the runtime

// ── PyO3 ─────────────────────────────────────────────────────────────
#[pyfunction] fn f(py: Python<'_>, a: Vec<f32>) -> PyResult<f32> { Ok(py.detach(|| work(&a))) }
#[pyclass] struct Index { }      #[pymethods] impl Index { #[new] fn new() -> Self { } }
#[pymodule] fn mymod(m: &Bound<'_, PyModule>) -> PyResult<()> { m.add_function(wrap_pyfunction!(f, m)?) }
```

(The cheat sheet is a reference card, not one runnable program.)

**Commands**

```bash
cargo new app            cargo add serde --features derive      cargo check
cargo run --release      cargo test                             cargo clippy
cargo fmt                cargo doc --open                       cargo build --release
maturin develop --release                                       rustc --explain E0382
```

**The ten sentences to remember**

1. One owner per value; when the owner dies, the value is freed.
2. Many readers or one writer — never both.
3. A reference can never outlive what it points to; lifetimes are how the compiler checks that.
4. `String` owns, `&str` borrows. `Vec<T>` owns, `&[T]` borrows.
5. No `null`: `Option`. No exceptions: `Result` and `?`.
6. `enum` + `match` = every case handled, checked by the compiler.
7. Traits are the contract; generics are free at runtime, `dyn` costs one jump.
8. No GIL: `Send` and `Sync` make data races a compile error.
9. Async is for waiting, threads are for computing; never block the runtime.
10. Python on top, Rust underneath: PyO3, NumPy without copies, release the GIL.

---

*Tested with rustc 1.97 (edition 2024), PyO3 0.30, axum 0.8, tokio 1.54, candle 0.11, burn 0.22, polars 0.55 — November 2026. Stack this with your `PYTHON_MASTERY.md`: Python for the idea, Rust for the engine.*
