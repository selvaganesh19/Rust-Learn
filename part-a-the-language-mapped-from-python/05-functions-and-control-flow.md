[← 4. Strings — `String` vs `&str`](../part-a-the-language-mapped-from-python/04-strings-string-vs-str.md) · [Index](../README.md) · [6. Collections — Vec, HashMap, HashSet, tuple →](../part-a-the-language-mapped-from-python/06-collections-vec-hashmap-hashset-tuple.md)

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

[← 4. Strings — `String` vs `&str`](../part-a-the-language-mapped-from-python/04-strings-string-vs-str.md) · [Index](../README.md) · [6. Collections — Vec, HashMap, HashSet, tuple →](../part-a-the-language-mapped-from-python/06-collections-vec-hashmap-hashset-tuple.md)
