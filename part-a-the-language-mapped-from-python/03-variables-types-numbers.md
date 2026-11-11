[← 2. Python → Rust Rosetta table](../part-a-the-language-mapped-from-python/02-python-rust-rosetta-table.md) · [Index](../README.md) · [4. Strings — `String` vs `&str` →](../part-a-the-language-mapped-from-python/04-strings-string-vs-str.md)

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

[← 2. Python → Rust Rosetta table](../part-a-the-language-mapped-from-python/02-python-rust-rosetta-table.md) · [Index](../README.md) · [4. Strings — `String` vs `&str` →](../part-a-the-language-mapped-from-python/04-strings-string-vs-str.md)
