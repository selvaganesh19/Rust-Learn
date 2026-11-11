# Part A — The language, mapped from Python

[← Back to index](../README.md)

**In this module**

- [0. Why Rust for AI — the honest picture](#0-why-rust-for-ai--the-honest-picture)
- [1. Setup and Cargo](#1-setup-and-cargo)
- [2. Python → Rust Rosetta table](#2-python--rust-rosetta-table)
- [3. Variables, types, numbers](#3-variables-types-numbers)
- [4. Strings — `String` vs `&str`](#4-strings--string-vs-str)
- [5. Functions and control flow](#5-functions-and-control-flow)
- [6. Collections — Vec, HashMap, HashSet, tuple](#6-collections--vec-hashmap-hashset-tuple)
- [7. Structs and `impl` — the class replacement](#7-structs-and-impl--the-class-replacement)
- [8. Enums and `match` — Rust's superpower](#8-enums-and-match--rusts-superpower)
- [9. `Option` and `Result` — no None bugs, no exceptions](#9-option-and-result--no-none-bugs-no-exceptions)
- [10. Traits and generics](#10-traits-and-generics)
- [11. Closures and iterators](#11-closures-and-iterators)
- [12. Modules, crates, tests](#12-modules-crates-tests)

---

## 0. Why Rust for AI — the honest picture

**In one line:** Python is the steering wheel of AI, Rust is slowly becoming the engine under it.

You will not train GPT in Rust. Research and training stay in Python + PyTorch. Rust wins in the places where Python is slow or unsafe: data loading, tokenising, serving, vector search, tooling.

You are already running Rust every day without knowing it:

| You type in Python | What actually runs |
|---|---|
| `from tokenizers import Tokenizer` (Hugging Face) | Rust core, Python wrapper (PyO3) |
| `import polars as pl` | Rust DataFrame engine |
| `from pydantic import BaseModel` (v2) | `pydantic-core`, written in Rust |
| `import orjson` | Rust JSON parser |
| `safetensors` model files | Rust library |
| `uv pip install`, `ruff check` | Both tools are Rust binaries |
| Qdrant, LanceDB, Meilisearch | Databases written in Rust |

See the pattern: **Python API on top, Rust core below.** That exact skill — writing a Rust core and wrapping it for Python — is Part D of this file, and very few Python developers have it.

**Where Rust fits in an AI system**

```mermaid
flowchart LR
    A[Raw data] -->|polars, arrow| B[Clean data]
    B -->|tokenizers| C[Token IDs]
    C -->|PyTorch in Python| D[Trained model]
    D -->|safetensors / ONNX| E[Model file]
    E -->|candle, ort, burn| F[Inference in Rust]
    F -->|axum, tokio| G[API server]
    G -->|qdrant, lancedb| H[Vector search / RAG]
```

Only the training box in the middle is firmly Python. Everything left and right of it is where Rust is used in production.

**Why Rust and not C++ for this job**

| Problem | Python | C++ | Rust |
|---|---|---|---|
| Speed | Slow interpreter | Fast | Fast (same LLVM backend as Clang) |
| Use all CPU cores | GIL blocks threads in the standard build | Yes, but data races are your problem | Yes, and data races are compile errors |
| Memory bugs (use-after-free, double free) | Not possible | Common, found at 2 a.m. in production | Rejected by the compiler in safe code |
| `None` crash | `AttributeError` at runtime | Null pointer crash | `Option<T>` forces you to handle it |
| Packaging | pip, venv, wheels | CMake pain | `cargo` — one tool |

**Say this in an interview:** "I use Python for experiments and Rust for the hot path. Rust gives C-level speed with memory safety checked at compile time and no GIL, so one process can use every core. Most fast Python AI libraries — tokenizers, polars, pydantic-core — already follow this design."

---

## 1. Setup and Cargo

**In one line:** `cargo` is pip + venv + pytest + black + build tool in one command.

```bash
# install (Linux / macOS). On Windows download rustup-init.exe from rustup.rs
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

rustc --version          # compiler
cargo --version          # build tool + package manager
rustup update stable     # upgrade Rust (a new stable version comes every 6 weeks)

cargo new hello          # create project (binary)
cd hello
cargo run                # compile + run (debug build, slow code, fast compile)
cargo run --release      # optimised build — ALWAYS use this when you measure speed
```

**Project layout**

```
hello/
├── Cargo.toml        # like pyproject.toml — name, version, dependencies
├── Cargo.lock        # like poetry.lock / uv.lock — exact versions
├── src/
│   └── main.rs       # entry point: fn main()
└── target/           # build output (like __pycache__, never commit it)
```

**Daily commands, mapped from Python**

| You do in Python | You do in Rust |
|---|---|
| `python main.py` | `cargo run` |
| `pip install requests` | `cargo add reqwest` |
| `pip install "uvicorn[standard]"` (extras) | `cargo add tokio --features full` (features) |
| `python -m venv .venv` | Not needed. Every project has its own dependencies. |
| `pytest` | `cargo test` |
| `black .` | `cargo fmt` |
| `ruff check` / `pylint` | `cargo clippy` |
| `mypy .` | `cargo check` (type check only, very fast) |
| `python -m pydoc` | `cargo doc --open` |
| PyPI | crates.io (docs for every crate at docs.rs) |

**Editor:** VS Code + the `rust-analyzer` extension. It shows types and errors as you type. Do not learn Rust without it.

**Trap:** beginners benchmark `cargo run` (debug build) and say "Rust is slow". Debug builds have zero optimisation and can be 10–100x slower. Measure only with `--release`.

---

## 2. Python → Rust Rosetta table

Keep this table open while reading the rest. Left side is from your Python files.

| Python | Rust | Note |
|---|---|---|
| `x = 10` | `let x = 10;` | Immutable by default |
| `x = 10; x += 1` | `let mut x = 10; x += 1;` | `mut` must be written |
| `int` (unlimited) | `i32`, `i64`, `u8`, `usize`… | Fixed size, can overflow |
| `float` | `f64` (and `f32` for ML) | |
| `bool` | `bool` | Not a number: `true + 1` is an error |
| `str` | `String` (owned) / `&str` (borrowed) | Section 4 |
| `None` | `None` inside `Option<T>` | Section 9 |
| `list` | `Vec<T>` | One element type only |
| `tuple` | `(i32, String)` | Fixed length, mixed types |
| `dict` | `HashMap<K, V>` | Not ordered. `BTreeMap` is sorted. |
| `set` | `HashSet<T>` | |
| `collections.deque` | `VecDeque<T>` | |
| `heapq` | `BinaryHeap<T>` | Max-heap by default (Python is min-heap) |
| `def f(a: int) -> int:` | `fn f(a: i32) -> i32 { }` | Types are compulsory in signatures |
| `lambda x: x + 1` | `\|x\| x + 1` | Closure |
| `class` | `struct` + `impl` | No inheritance |
| `ABC` / `Protocol` | `trait` | |
| `__str__`, `__eq__`, `__add__` | `Display`, `PartialEq`, `Add` traits | Section 10 |
| `try / except` | `Result<T, E>` + `?` | No exceptions |
| `raise ValueError("x")` | `return Err(...)` | |
| `with open(...) as f:` | Just a scope `{ }` — `Drop` closes it | Section 20 |
| `[x*2 for x in v if x > 0]` | `v.iter().filter(\|x\| **x > 0).map(\|x\| x * 2).collect()` | Section 11 |
| generator / `yield` | `Iterator` trait | Lazy, same idea |
| `import os` | `use std::fs;` | |
| `if __name__ == "__main__":` | `fn main() { }` | |
| `print(f"{x=}")` | `println!("{x:?}");` or `dbg!(x);` | |
| `assert x > 0` | `assert!(x > 0);` | |
| `pass` / `...` | `todo!()` or `{}` | `todo!()` compiles but panics if reached |
| `async def` + `await x` | `async fn` + `x.await` | Section 22 |
| `threading.Lock` | `Mutex<T>` | The lock *owns* the data |
| GIL | Does not exist | Section 21 |

---

## 3. Variables, types, numbers

**In one line:** every value has a fixed type known at compile time, and nothing changes unless you write `mut`.

**Python you know:** `x = 10` then `x = "hello"` is fine. Type lives on the object and is checked while running.

```rust
fn main() {
    let x = 10;              // immutable, type inferred as i32
    // x = 11;               // error[E0384]: cannot assign twice to immutable variable

    let mut count = 0;       // `mut` = "I am going to change this"
    count += 1;

    let x = x * 2;           // shadowing: a NEW variable that reuses the name
    let x = x.to_string();   // shadowing can even change the type (i32 -> String)
    println!("{x} {count}");

    const MAX_TOKENS: usize = 4096;    // compile-time constant, type is compulsory
    let price: f64 = 19.99;
    let big = 10_000_001_u64;          // type suffix + underscores for reading
    let ok = true;
    let ch = 'அ';                      // char = 4 bytes, one Unicode scalar value
    println!("{MAX_TOKENS} {price} {big} {ok} {ch}");

    // Numbers never convert silently
    let a: i32 = 7;
    let b: f64 = a as f64 / 2.0;       // `as` = explicit cast
    let c: i64 = a.into();             // lossless widening
    let d = u8::try_from(300_i32);     // checked narrowing -> Err, 300 does not fit in u8
    println!("{b} {c} {d:?}");

    // Overflow: Python ints grow forever. Rust ints have a fixed size.
    let m: u8 = 250;
    println!("{:?}", m.checked_add(10));    // None      -> you decide what to do
    println!("{}", m.wrapping_add(10));     // 4         -> wraps around
    println!("{}", m.saturating_add(10));   // 255       -> stops at max

    // Division is different from Python for negative numbers
    println!("{} {} {}", -7 / 2, -7 % 2, (-7_i32).rem_euclid(2));   // -3 -1 1
    println!("{}", 0.1 + 0.2 == 0.3);       // false — same IEEE 754 floats as Python
    println!("{}", 2_i32.pow(10));          // 1024 (no ** operator)
}
```

**The number types**

| Type | Size | Use it for |
|---|---|---|
| `i8 i16 i32 i64 i128` | 1–16 bytes | Signed integers. `i32` is the default. |
| `u8 u16 u32 u64 u128` | 1–16 bytes | Unsigned. `u8` = one byte, `u32` = token IDs. |
| `usize` / `isize` | Pointer size (8 bytes on 64-bit) | Indexes and lengths. `v[i]` needs `usize`. |
| `f32` | 4 bytes | Model weights, embeddings (half the memory of `f64`) |
| `f64` | 8 bytes | Default float, same as Python `float` |
| `bool` | 1 byte | |
| `char` | 4 bytes | One Unicode scalar value |
| `()` | 0 bytes | "Unit" — what a function returns when it returns nothing |

**Inside the machine:** an `i32` is 4 raw bytes on the stack. No object header, no reference count, no pointer. A Python `int` is a heap object of 28 bytes or more. A `Vec<f32>` of one million numbers is exactly 4 MB of continuous memory; a Python list of one million floats is 8 MB of pointers plus 24 MB of float objects scattered on the heap. This is the first reason Rust numeric code is fast: the CPU cache loves continuous memory.

**Trap — overflow:** `let x: u8 = 255; x + 1` panics in a debug build and silently wraps to `0` in a release build. If overflow is possible, write `checked_add`, `wrapping_add` or `saturating_add` and make the choice visible.

**Trap — `-7 / 2`:** Python gives `-4` (floor). Rust gives `-3` (toward zero, like C). Use `div_euclid` / `rem_euclid` when you need Python behaviour.

**Say this in an interview:** "Shadowing creates a new binding and can change the type; `mut` changes the value in the same binding. I prefer shadowing for step-by-step transforms because the old value becomes unreachable."

---

## 4. Strings — `String` vs `&str`

**In one line:** `String` owns its text on the heap; `&str` is a borrowed window (pointer + length) into text owned by someone else.

**Python you know:** one `str` type, immutable, indexing gives a character.

Think of a hostel room. `String` is the student who has the room — can move furniture, can vacate it. `&str` is a visitor's pass — can look, cannot change, and is valid only while the room exists.

```rust
fn main() {
    let literal: &str = "hello";                    // text baked into the program binary
    let mut owned: String = String::from("hello");  // heap buffer, growable
    owned.push_str(" world");
    owned.push('!');

    let view: &str = &owned[0..5];                  // slice: pointer + length, nothing copied
    println!("{view} | {owned} | {}", literal.len());

    // Converting
    let s1 = "abc".to_string();        // &str -> String (allocates)
    let s2: String = "abc".into();     // same thing
    let s3 = format!("{s1}-{s2}-{}", 42);   // f-string that returns a String
    let r: &str = s3.as_str();         // String -> &str (free)
    println!("{r}");

    // UTF-8: len() is BYTES, not characters
    let tamil = "வணக்கம்";
    println!("bytes={} chars={}", tamil.len(), tamil.chars().count());   // bytes=21 chars=7
    // let c = tamil[0];               // error[E0277]: `str` cannot be indexed by `{integer}`
    println!("{:?}", tamil.chars().next());          // Some('வ')

    // The methods you used in Python
    let s = "  Hello World Python  ";
    println!("{}", s.trim().to_uppercase());                     // .strip().upper()
    println!("{:?}", s.split_whitespace().collect::<Vec<_>>());  // .split()
    println!("{}", s.trim().replace("Hello", "Hi"));             // .replace()
    println!("{} {:?}", s.contains("World"), s.find("World"));   // `in`, .find()
    println!("{}", s.trim().chars().rev().collect::<String>());  // s[::-1]
    println!("{}", ["a", "b", "c"].join(","));                   // ",".join(...)
    println!("{:?} {}", "42".parse::<i32>(), "4x".parse::<i32>().is_err());   // int("42")

    // Formatting — almost the same mini-language as f-strings
    let (name, bal) = ("Selva", 12345.678);
    println!("{name:>10}|{bal:.2}|{:05}|{:#x}|{:?}", 42, 255, "dbg");
}
```

**Which one to use**

| Situation | Use | Why |
|---|---|---|
| Function parameter that only reads text | `&str` | Accepts both `&String` and literals, no copy |
| Struct field that must keep the text | `String` | The struct owns it, no lifetime headache |
| Returning newly built text | `String` | Caller becomes the owner |
| Raw bytes (file, network) | `Vec<u8>` / `&[u8]` | Not guaranteed to be UTF-8 |
| File paths | `PathBuf` / `&Path` | Paths are not always valid UTF-8 |

**Inside the machine**

```
let owned = String::from("hello");

 STACK (24 bytes)                 HEAP
┌──────────┐                    ┌───┬───┬───┬───┬───┐
│ ptr    ──┼──────────────────► │ h │ e │ l │ l │ o │
│ len = 5  │                    └───┴───┴───┴───┴───┘
│ cap = 5  │                      ▲
└──────────┘                      │
                                  │
let view: &str = &owned[1..4];    │   (16 bytes on stack)
┌──────────┐                      │
│ ptr    ──┼────────────── points at 'e'
│ len = 3  │
└──────────┘
```

`String` is three machine words: pointer, length, capacity. `&str` is two: pointer, length. A slice costs nothing — this is why Rust parsers (tokenizers, JSON) are fast: they cut the input into thousands of `&str` pieces without copying one byte.

**Trap — indexing:** `s[0]` does not compile, because one "character" can be 1 to 4 bytes in UTF-8, so `s[0]` cannot be O(1). Use `s.chars().nth(0)` for characters, `s.as_bytes()[0]` for bytes. Slicing `&s[0..2]` uses byte positions and **panics** if you cut a character in the middle.

**Trap — `+` in a loop:** same lesson as your Python file (Q6). `s = s + "x"` works but prefer `s.push_str("x")`, or `String::with_capacity(n)` when you know the size.

**Say this in an interview:** "`String` is an owned, growable UTF-8 buffer with pointer, length and capacity. `&str` is a fat pointer — pointer and length — into UTF-8 bytes it does not own. APIs should take `&str` and return `String`."

---

## 5. Functions and control flow

**In one line:** almost everything is an *expression* that produces a value — `if`, `match`, `loop`, even a `{ }` block.

**Python you know:** `label = "pos" if x > 0 else "neg"`, `for i in range(5)`, `for-else`, and `return` everywhere.

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b              // last line WITHOUT semicolon = the return value
}

fn grade(marks: u32) -> &'static str {
    match marks {      // your if/elif chain from Q49, as a range match
        90..=100 => "A+",
        80..=89 => "A",
        70..=79 => "B",
        60..=69 => "C",
        50..=59 => "D",
        _ => "F",      // `_` = everything else. Remove it and the code will not compile.
    }
}

fn main() {
    let x = 42;
    let label = if x > 0 { "positive" } else { "non-positive" };   // if is an expression

    for i in 0..5 {                          // range(5)
        print!("{i} ");
    }
    for i in (1..=10).rev().step_by(3) {     // range(10, 0, -3) -> 10 7 4 1
        print!("{i} ");
    }
    println!();

    let langs = ["python", "rust", "go"];
    for (i, lang) in langs.iter().enumerate() {      // enumerate()
        println!("{i}: {lang}");
    }

    let mut n = 27;
    let mut steps = 0;
    while n != 1 {
        n = if n % 2 == 0 { n / 2 } else { 3 * n + 1 };
        steps += 1;
    }

    // `loop` = while True, and `break` can carry a value out
    let mut counter = 0;
    let found = loop {
        counter += 7;
        if counter % 5 == 0 {
            break counter;
        }
    };

    // Labelled break: exit the OUTER loop from the inner one
    let mut pair = None;
    'outer: for a in 1..10 {
        for b in 1..10 {
            if a * b == 42 {
                pair = Some((a, b));
                break 'outer;
            }
        }
    }

    // A block is an expression too
    let area = {
        let w = 3;
        let h = 4;
        w * h          // no semicolon -> value of the block
    };

    println!("{label} {steps} {found} {pair:?} {area} {} {}", add(2, 3), grade(85));
}
```

**What Rust does not have (and what to use)**

| Python feature | Rust way |
|---|---|
| Default arguments `def f(x, k=5)` | `Option<usize>` parameter, or a builder struct, or two functions |
| Keyword arguments | A struct for the options: `search(Query { k: 5, ..Default::default() })` |
| `*args` | A slice `&[T]`. Macros like `println!` are the only variadic things. |
| `for … else` | `iter().find(...)`, `iter().any(...)`, or a labelled break |
| Truthiness `if items:` | Be explicit: `if !items.is_empty()` |
| Chained compare `0 <= x < 10` | `(0..10).contains(&x)` |
| Ternary `a if c else b` | `if c { a } else { b }` |

**Inside the machine:** a function call in Rust is a plain machine `call` instruction, and small functions are inlined by the optimiser — the call disappears completely. In CPython every call sets up an interpreter frame and goes through the bytecode loop. This is why "many tiny functions" is free in Rust and expensive in Python.

**Trap — the semicolon:** `a + b` returns the value. `a + b;` throws the value away and returns `()`. The compiler error will say `expected i32, found ()` — when you see that, look for an extra semicolon on the last line.

---

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

## 7. Structs and `impl` — the class replacement

**In one line:** data goes in a `struct`, behaviour goes in an `impl` block, and the first parameter tells exactly what the method is allowed to do with the object.

**Python you know:** `class BankAccount` with `__init__` and `self` (your Q12).

```rust
use std::fmt;

#[derive(Debug, Clone, PartialEq)]      // auto-generate __repr__, copy(), __eq__
struct Account {
    owner: String,
    balance: f64,
}

impl Account {
    // No __init__. A constructor is just a normal function that returns Self.
    fn new(owner: &str, balance: f64) -> Self {
        Self { owner: owner.to_string(), balance }
    }

    fn balance(&self) -> f64 {            // &self      -> read only
        self.balance
    }

    fn deposit(&mut self, amount: f64) {  // &mut self  -> may change the object
        self.balance += amount;
    }

    fn close(self) -> f64 {               // self       -> consumes the object
        self.balance                      // after this call the account does not exist
    }
}

impl fmt::Display for Account {           // __str__
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}: ₹{:.2}", self.owner, self.balance)
    }
}

fn transfer(from: &mut Account, to: &mut Account, amount: f64) -> bool {
    if from.balance < amount {
        return false;
    }
    from.balance -= amount;
    to.balance += amount;
    true
}

fn main() {
    let mut selva = Account::new("Selva", 1000.0);
    let mut priya = Account::new("Priya", 500.0);

    selva.deposit(200.0);
    let ok = transfer(&mut selva, &mut priya, 300.0);

    println!("{selva} | {priya} | ok={ok}");      // Display
    println!("{selva:?}");                         // Debug
    println!("{}", selva == selva.clone());        // PartialEq + Clone

    let final_amount = priya.close();
    // println!("{priya}");     // error[E0382]: borrow of moved value: `priya`
    println!("{final_amount} {}", selva.balance());

    // Struct update syntax, tuple struct, unit struct
    let guest = Account { owner: "Guest".into(), ..selva.clone() };
    struct Meters(f64);                            // "newtype": a f64 that cannot be mixed with others
    let height = Meters(1.75);
    println!("{} {}", guest.owner, height.0);
}
```

**`self` in three flavours — this has no Python equivalent**

| Signature | Meaning | Python feeling |
|---|---|---|
| `fn f(&self)` | Borrow, read only | A getter |
| `fn f(&mut self)` | Borrow, may modify. Caller's variable must be `mut`. | A normal method |
| `fn f(self)` | Takes ownership. Object is gone after the call. | No equivalent |
| `fn f() -> Self` | No `self`: associated function, called as `Type::f()` | `@classmethod` / `@staticmethod` |

The `close(self)` trick is powerful: you can make "use after close" a **compile error**. In Python you would write `if self.closed: raise ...` and hope a test catches it.

**Python class feature → Rust**

| Python | Rust |
|---|---|
| `__init__` | `fn new(...) -> Self` (only a convention) |
| `@dataclass` | `#[derive(Debug, Clone, PartialEq)]` |
| `@dataclass(frozen=True)` | Default behaviour — nothing is mutable without `mut` |
| `_private` (convention only) | Fields are private outside the module unless you write `pub` — enforced |
| Class attribute | `const` inside `impl`, or a `static` |
| `@property` | A method `fn name(&self) -> &str` |
| Inheritance | Does not exist. Use composition + traits (Section 10). |

**Inside the machine:** a struct is just its fields laid side by side — no `__dict__`, no object header. `Account` is 32 bytes: 24 for the `String` and 8 for the `f64`. Methods are ordinary functions; `selva.deposit(200.0)` compiles to `Account::deposit(&mut selva, 200.0)` — a direct call, no attribute lookup. This is like your `__slots__` discussion (Q40), except it is always on.

**Trap — no inheritance:** do not try to rebuild `class Dog(Animal)`. Put shared behaviour in a trait, shared data in a field. "Composition over inheritance" is not advice in Rust, it is the only option.

---

## 8. Enums and `match` — Rust's superpower

**In one line:** a Rust enum is "this value is exactly one of these shapes, and each shape can carry its own data" — and `match` forces you to handle every shape.

**Python you know:** `Enum` is only names. For "one of several shapes" you used dicts with a `"type"` key, or `match/case` on dicts (your Q50) — but nothing stops you from forgetting a case.

```rust
#[derive(Debug)]
enum Payment {
    Cash,                                   // no data
    Upi { id: String },                     // named fields
    Card { last4: String, emi_months: u8 }, // different fields
}

fn fee(p: &Payment, amount: f64) -> f64 {
    match p {
        Payment::Cash | Payment::Upi { .. } => 0.0,
        Payment::Card { emi_months: 0, .. } => amount * 0.02,
        Payment::Card { emi_months, .. } => amount * 0.02 + f64::from(*emi_months) * 10.0,
    }
    // Add a new variant `Wallet` later -> this function stops compiling
    // until you handle it. The compiler finds every place you forgot.
}

fn describe(n: i32) -> String {
    match n {
        0 => "zero".to_string(),
        x if x < 0 => format!("negative {x}"),        // guard
        1 | 2 | 3 => "small".to_string(),             // or-pattern
        big @ 100.. => format!("big number {big}"),   // bind with @
        _ => "medium".to_string(),
    }
}

fn main() {
    let payments = vec![
        Payment::Cash,
        Payment::Upi { id: "selva@okaxis".into() },
        Payment::Card { last4: "1234".into(), emi_months: 0 },
        Payment::Card { last4: "9876".into(), emi_months: 6 },
    ];
    for p in &payments {
        println!("{p:?} -> fee ₹{}", fee(p, 1000.0));
    }

    // if let: care about only one variant
    if let Payment::Upi { id } = &payments[1] {
        println!("upi id = {id}");
    }

    // let-else: take the value or leave early
    let Payment::Card { last4, .. } = &payments[2] else {
        panic!("expected a card");
    };
    println!("card ending {last4}");

    // matches! gives a bool
    let cards = payments.iter().filter(|p| matches!(p, Payment::Card { .. })).count();

    // Destructure tuples and structs
    let point = (3, -3);
    let place = match point {
        (0, 0) => "origin",
        (x, y) if x == -y => "anti-diagonal",
        (_, 0) => "on x axis",
        _ => "somewhere",
    };

    println!("{cards} {place} {} {} {}", describe(-5), describe(2), describe(500));
}
```

**Why AI code loves enums:** real data comes in shapes. A chat message is `System`, `User`, `Assistant` or `ToolCall { name, args }`. A tensor's device is `Cpu`, `Cuda(index)` or `Metal`. A job is `Queued`, `Running { started }`, `Done(result)` or `Failed(error)`. With an enum, "Done but no result" or "Failed with a result" simply cannot be written. People call this **making illegal states unrepresentable**, and it removes a whole family of bugs before the program runs.

**Inside the machine:** an enum is stored as a small tag (which variant) plus enough space for the biggest variant — like a C tagged union, but safe. `match` compiles to a jump table or a few comparisons, never a dictionary lookup. No heap allocation happens unless a variant itself holds a `String` or `Vec`.

**Trap — `_` hides future bugs:** a catch-all `_ =>` arm means the compiler will stay quiet when someone adds a new variant. In code you own, list every variant; keep `_` for open sets like integers.

**Say this in an interview:** "Rust enums are algebraic data types — sum types. Together with exhaustive `match`, the compiler proves I handled every case, so adding a variant gives me a to-do list of compile errors instead of a production bug."

---

## 9. `Option` and `Result` — no None bugs, no exceptions

**In one line:** "may be missing" is `Option<T>`, "may fail" is `Result<T, E>` — both are ordinary enums, so the type of a function tells the full truth and the compiler makes you deal with it.

**Python you know:** any variable can secretly be `None` (`AttributeError: 'NoneType' has no attribute...`), and any function can raise anything without saying so.

```rust
enum Option<T> { Some(T), None }       // this is the whole definition
enum Result<T, E> { Ok(T), Err(E) }
```

```rust
use std::collections::HashMap;
use std::fmt;
use std::num::ParseIntError;

// Custom error = an enum. One variant per way of failing.
#[derive(Debug)]
enum ConfigError {
    Missing(String),
    BadNumber { key: String, source: ParseIntError },
    OutOfRange(u32),
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ConfigError::Missing(k) => write!(f, "missing key `{k}`"),
            ConfigError::BadNumber { key, source } => write!(f, "`{key}` is not a number: {source}"),
            ConfigError::OutOfRange(n) => write!(f, "port {n} is out of range"),
        }
    }
}

impl std::error::Error for ConfigError {}

fn get_port(cfg: &HashMap<&str, &str>) -> Result<u16, ConfigError> {
    // `?` = "if this is an Err, return it from the function right now; otherwise unwrap it"
    let raw = cfg.get("port").ok_or_else(|| ConfigError::Missing("port".into()))?;
    let n: u32 = raw
        .trim()
        .parse()
        .map_err(|e| ConfigError::BadNumber { key: "port".into(), source: e })?;
    u16::try_from(n).map_err(|_| ConfigError::OutOfRange(n))
}

fn main() {
    // ── Option ──────────────────────────────────────────────────────
    let names = vec!["asha", "ravi"];
    let third: Option<&&str> = names.get(2);                    // None, not a crash

    let shout = names.get(0).map(|s| s.to_uppercase());         // transform if present
    let fallback = names.get(5).copied().unwrap_or("nobody");   // default value
    let long = names.iter().find(|n| n.len() > 10).is_some();

    match third {
        Some(name) => println!("third is {name}"),
        None => println!("no third person"),
    }
    if let Some(s) = &shout {
        println!("{s} {fallback} {long}");
    }

    // ── Result ──────────────────────────────────────────────────────
    let good = HashMap::from([("port", " 8080 ")]);
    let empty: HashMap<&str, &str> = HashMap::new();
    let text = HashMap::from([("port", "eighty")]);
    let huge = HashMap::from([("port", "99999")]);

    for cfg in [&good, &empty, &text, &huge] {
        match get_port(cfg) {
            Ok(port) => println!("listening on {port}"),
            Err(e) => println!("config error: {e}"),
        }
    }

    // Quick ways out (fine in scripts and tests, think twice in a server)
    let port = get_port(&good).unwrap();                   // panic on Err
    let port2 = get_port(&empty).unwrap_or(3000);          // default on Err
    let port3 = get_port(&good).expect("port must be set"); // panic with your message
    println!("{port} {port2} {port3}");
}
```

**Python → Rust for errors**

| Python | Rust |
|---|---|
| `return None` | `None` (function returns `Option<T>`) |
| `if x is not None:` | `if let Some(x) = x { }` |
| `x or default` | `x.unwrap_or(default)` |
| `x.attr if x else None` | `x.map(\|v\| v.attr)` |
| `raise ValueError(msg)` | `return Err(MyError::Bad(msg))` |
| `try: ... except E:` | `match f() { Ok(v) => ..., Err(e) => ... }` |
| Let the exception bubble up | `f()?` |
| `raise X from e` | `.map_err(...)`, or `#[from]` / `.context(...)` below |
| `finally:` | `Drop` (Section 20) |
| Crash on a bug (`assert`) | `panic!`, `unwrap()`, `expect()` |

**In real projects nobody hand-writes `Display` for errors.** Two crates do it:

- **`thiserror`** — for libraries. Derives the error enum boilerplate.
- **`anyhow`** — for applications. One error type that holds anything, plus `.context("what I was doing")`.

```rust
// cargo add thiserror anyhow
use anyhow::{Context, Result};
use thiserror::Error;

#[derive(Debug, Error)]
enum ModelError {
    #[error("model file not found: {0}")]
    NotFound(String),
    #[error("wrong shape: expected {expected}, got {got}")]
    Shape { expected: usize, got: usize },
    #[error(transparent)]
    Io(#[from] std::io::Error),       // `?` converts io::Error into ModelError::Io automatically
}

fn load(path: &str) -> std::result::Result<Vec<u8>, ModelError> {
    if !std::path::Path::new(path).exists() {
        return Err(ModelError::NotFound(path.into()));
    }
    let bytes = std::fs::read(path)?;
    if bytes.len() < 8 {
        return Err(ModelError::Shape { expected: 8, got: bytes.len() });
    }
    Ok(bytes)
}

fn run() -> Result<()> {
    let bytes = load("model.safetensors").context("while starting the inference server")?;
    println!("{} bytes", bytes.len());
    Ok(())
}

fn main() {
    if let Err(e) = run() {
        eprintln!("error: {e:#}");
        // error: while starting the inference server: model file not found: model.safetensors
    }
}
```

**Inside the machine:** `Result` is a normal return value — a tag plus data, returned in registers or on the stack. There is no stack unwinding, no hidden jump. The success path costs one branch. `Option<&T>` and `Option<Box<T>>` are the same size as the raw pointer: the compiler uses the "impossible" null value to mean `None` (niche optimisation, measured in Section 18). So Rust's safe "nullable" costs nothing extra.

**`panic!` is not an exception.** A panic means "this is a bug, the program cannot continue": index out of range, `unwrap()` on `None`, a failed `assert!`. Do not use panics for things that happen in normal life (file missing, bad user input) — that is what `Result` is for.

**Trap — `unwrap()` everywhere:** it compiles, it works on your laptop, and it kills the server on the first bad request. Rule: `unwrap()` in tests and throw-away scripts; `?` in real code; `expect("reason")` only when you can explain why it can never fail.

**Say this in an interview:** "Errors are values. `Result` puts failure in the type signature, `?` propagates it in one character, and there is no hidden control flow. I use `thiserror` for library error enums and `anyhow` with `.context()` in application code."

---

## 10. Traits and generics

**In one line:** a trait is a named set of methods a type promises to have — like `Protocol`/`ABC` — and generics let you write one function that works for every type that keeps the promise, with zero runtime cost.

**Python you know:** duck typing ("if it has `.charge()`, call it", your Q18), `ABC`, `Protocol`, and dunder methods.

```rust
use std::fmt::Display;
use std::ops::Add;

// ── The contract ────────────────────────────────────────────────────
trait Embedder {
    fn dim(&self) -> usize;
    fn embed(&self, text: &str) -> Vec<f32>;

    // Default method: every implementor gets it free, and may override it
    fn embed_batch(&self, texts: &[&str]) -> Vec<Vec<f32>> {
        texts.iter().map(|t| self.embed(t)).collect()
    }
}

// ── Two unrelated types keep the promise ────────────────────────────
struct LengthEmbedder;
struct ConstantEmbedder {
    value: f32,
    dim: usize,
}

impl Embedder for LengthEmbedder {
    fn dim(&self) -> usize {
        2
    }
    fn embed(&self, text: &str) -> Vec<f32> {
        vec![text.len() as f32, text.split_whitespace().count() as f32]
    }
}

impl Embedder for ConstantEmbedder {
    fn dim(&self) -> usize {
        self.dim
    }
    fn embed(&self, _text: &str) -> Vec<f32> {
        vec![self.value; self.dim]
    }
}

// ── Static dispatch: the compiler writes one copy of this per type ──
fn show<E: Embedder>(model: &E, text: &str) {
    println!("{:?}", model.embed(text));
}

// Same thing, shorter spelling
fn show_batch(model: &impl Embedder, texts: &[&str]) {
    println!("{:?}", model.embed_batch(texts));
}

// ── Dynamic dispatch: decided while running, like Python ────────────
fn run_all(models: &[Box<dyn Embedder>], text: &str) {
    for m in models {
        println!("dim {} -> {:?}", m.dim(), m.embed(text));
    }
}

// ── Generic function with trait bounds ──────────────────────────────
fn largest<T: PartialOrd + Copy>(items: &[T]) -> Option<T> {
    let mut iter = items.iter().copied();
    let mut best = iter.next()?;          // empty slice -> return None
    for x in iter {
        if x > best {
            best = x;
        }
    }
    Some(best)
}

fn tag<T>(x: T) -> String
where
    T: Display,                           // `where` clause: same bounds, easier to read
{
    format!("<{x}>")
}

// ── Operator overloading = implementing a std trait ─────────────────
#[derive(Debug, Clone, Copy, PartialEq)]
struct Vec2 {
    x: f32,
    y: f32,
}

impl Add for Vec2 {                       // __add__
    type Output = Vec2;
    fn add(self, other: Vec2) -> Vec2 {
        Vec2 { x: self.x + other.x, y: self.y + other.y }
    }
}

impl From<(f32, f32)> for Vec2 {          // conversion: Vec2::from((1.0, 2.0)) or .into()
    fn from(t: (f32, f32)) -> Self {
        Vec2 { x: t.0, y: t.1 }
    }
}

fn main() {
    show(&LengthEmbedder, "rust is fast");
    show_batch(&ConstantEmbedder { value: 0.5, dim: 3 }, &["a", "b"]);

    let models: Vec<Box<dyn Embedder>> = vec![
        Box::new(LengthEmbedder),
        Box::new(ConstantEmbedder { value: 1.0, dim: 2 }),
    ];
    run_all(&models, "hello world");

    println!("{:?} {:?} {:?}", largest(&[3, 9, 4]), largest(&[1.5, 0.2]), largest::<i32>(&[]));
    println!("{} {}", tag(42), tag("hi"));

    let a = Vec2 { x: 1.0, y: 2.0 };
    let b: Vec2 = (0.5, 0.5).into();
    println!("{:?}", a + b);
}
```

**Your dunder methods → Rust traits**

| Python dunder | Rust trait | How you usually get it |
|---|---|---|
| `__repr__` | `Debug` | `#[derive(Debug)]` |
| `__str__` | `Display` | Write `impl fmt::Display` |
| `__eq__` | `PartialEq`, `Eq` | `#[derive(PartialEq, Eq)]` |
| `__hash__` | `Hash` | `#[derive(Hash)]` |
| `__lt__` etc. | `PartialOrd`, `Ord` | `#[derive(PartialOrd, Ord)]` |
| `__add__`, `__mul__` | `Add`, `Mul` (in `std::ops`) | Write `impl Add` |
| `__getitem__` | `Index`, `IndexMut` | Write `impl Index` |
| `__iter__` / `__next__` | `IntoIterator` / `Iterator` | Write `impl Iterator` |
| `__call__` | `Fn`, `FnMut`, `FnOnce` | Closures implement these |
| `__del__`, `__exit__` | `Drop` | Write `impl Drop` |
| `copy.copy` / `deepcopy` | `Copy` / `Clone` | `#[derive(Clone, Copy)]` |
| `__init__` with no args | `Default` | `#[derive(Default)]` |
| `int(x)`, `str(x)` conversions | `From` / `Into`, `FromStr` | Write `impl From<A> for B` |
| `__bool__`, `__len__` | No trait | Just methods `is_empty()`, `len()` |

**Static vs dynamic — pick one consciously**

| | `fn f<T: Trait>(x: &T)` / `impl Trait` | `fn f(x: &dyn Trait)` |
|---|---|---|
| Decided | At compile time | While running |
| Speed | Direct call, can be inlined | One pointer jump through a vtable |
| Binary size | One copy per type used | One copy |
| Mixed types in one `Vec` | Not possible | Yes: `Vec<Box<dyn Trait>>` |
| Use it for | Hot loops, numeric code | Plugins, lists of different things |

**Inside the machine:** generics are compiled by **monomorphisation** — `largest::<i32>` and `largest::<f64>` become two separate machine-code functions, each as fast as if you had written it by hand. That is the meaning of "zero-cost abstraction". Details and proof in Section 19.

**Trap — "trait is not implemented":** `error[E0277]: the trait bound ... is not satisfied` is the Rust form of Python's `AttributeError`, but at compile time. Read which trait is missing, then add a `#[derive(...)]` or a bound like `T: Clone`.

**Say this in an interview:** "Traits give me duck typing with a compile-time contract. With generics the compiler monomorphises — static dispatch, inlinable. With `dyn Trait` I get dynamic dispatch through a vtable when I need mixed types. Rust has no inheritance; I share behaviour with traits and data with composition."

---

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

## 12. Modules, crates, tests

**In one line:** a *crate* is one compiled unit (a library or a binary, like a PyPI package); a *module* is a namespace inside it (like a `.py` file); everything is private until you say `pub`.

**Python you know:** files are modules, folders with `__init__.py` are packages, `_name` is private only by convention (your Q37).

```
my_app/
├── Cargo.toml
└── src/
    ├── main.rs          # binary crate root — declares the modules
    ├── config.rs        # module `config`
    └── model/
        ├── mod.rs       # module `model` (like __init__.py)
        └── tokenizer.rs # module `model::tokenizer`
```

```rust
// In real projects `mod config;` loads src/config.rs.
// Here the modules are written inline so the block runs as one file.
mod config {
    pub struct Settings {
        pub model_name: String,     // pub field: readable from outside
        max_tokens: usize,          // private field: only this module can touch it
    }

    impl Settings {
        pub fn new(model_name: &str) -> Self {
            Settings { model_name: model_name.to_string(), max_tokens: default_limit() }
        }

        pub fn max_tokens(&self) -> usize {
            self.max_tokens
        }
    }

    fn default_limit() -> usize {   // private function
        4096
    }
}

mod model {
    pub mod tokenizer {
        pub fn count_tokens(text: &str) -> usize {
            text.split_whitespace().count()
        }
    }

    use crate::config::Settings;    // `crate::` = absolute path from the crate root

    pub fn fits(settings: &Settings, text: &str) -> bool {
        tokenizer::count_tokens(text) <= settings.max_tokens()
    }
}

use config::Settings;
use model::tokenizer::count_tokens;

fn main() {
    let s = Settings::new("mini-lm");
    // s.max_tokens = 10;           // error[E0616]: field `max_tokens` of struct `Settings` is private
    println!("{} {} {}", s.model_name, count_tokens("rust is fast"), model::fits(&s, "hello"));
}

// Tests live next to the code. Run with: cargo test
#[cfg(test)]
mod tests {
    use super::*;                   // `super::` = parent module

    #[test]
    fn counts_words() {
        assert_eq!(count_tokens("a b  c"), 3);
    }

    #[test]
    fn long_text_does_not_fit() {
        let s = Settings::new("tiny");
        let text = "word ".repeat(5000);
        assert!(!model::fits(&s, &text));
    }
}
```

**`Cargo.toml` — read it like `pyproject.toml`**

```toml
[package]
name = "my_app"
version = "0.1.0"
edition = "2024"            # language edition; old crates keep working with new compilers

[dependencies]
serde = { version = "1", features = ["derive"] }   # features = optional parts of a crate
tokio = { version = "1", features = ["full"] }
anyhow = "1"                # "1" means >=1.0.0 and <2.0.0

[dev-dependencies]          # only for tests and benches
criterion = "0.8"

[profile.release]
opt-level = 3
lto = "thin"                # link-time optimisation across crates
```

**Words you will hear**

| Word | Meaning |
|---|---|
| Crate | One library or binary. Unit of compilation and of publishing. |
| Package | A folder with a `Cargo.toml`. Holds one library crate and/or binaries. |
| Workspace | Several packages sharing one `Cargo.lock` and `target/` (monorepo). |
| Feature | A switch that turns optional code and dependencies on. |
| Edition | 2015, 2018, 2021, 2024. Small syntax changes; crates of different editions link together. |
| `pub(crate)` | Public inside this crate only. |
| Prelude | The few names imported automatically (`Vec`, `Option`, `String`…). |

**Inside the machine:** all modules of a crate are compiled together into one unit, and privacy is checked by the compiler — outside code *cannot* reach a private field, there is no `_name` back door like Python's name mangling. That lets a library author change internals freely, and lets `unsafe` code inside a module rely on invariants nobody outside can break.

**Trap — `mod` vs `use`:** `mod foo;` *declares* that a module exists (it pulls the file into the build, once, in its parent). `use foo::bar;` only makes a shorter name. Writing `use` without the `mod` declaration gives "unresolved import".
