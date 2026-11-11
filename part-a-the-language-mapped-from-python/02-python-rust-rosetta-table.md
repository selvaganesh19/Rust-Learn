[← 1. Setup and Cargo](../part-a-the-language-mapped-from-python/01-setup-and-cargo.md) · [Index](../README.md) · [3. Variables, types, numbers →](../part-a-the-language-mapped-from-python/03-variables-types-numbers.md)

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

[← 1. Setup and Cargo](../part-a-the-language-mapped-from-python/01-setup-and-cargo.md) · [Index](../README.md) · [3. Variables, types, numbers →](../part-a-the-language-mapped-from-python/03-variables-types-numbers.md)
