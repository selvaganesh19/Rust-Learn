[← 23. `unsafe` and FFI](../part-c-internals/23-unsafe-and-ffi.md) · [Index](../README.md) · [25. Performance checklist →](../part-c-internals/25-performance-checklist.md)

## 24. Macros

**In one line:** a macro is code that writes code at compile time; anything with a `!` or inside `#[...]` is one.

**Python you know:** decorators and metaclasses change behaviour at runtime (your Q23, Q47). Rust macros generate ordinary source code before type checking, so the result is fully checked and costs nothing at runtime.

| Kind | Looks like | Examples | Python feeling |
|---|---|---|---|
| Declarative | `name!(...)` | `println!`, `vec!`, `format!`, `assert_eq!`, `matches!` | A function with flexible syntax |
| Derive | `#[derive(X)]` | `Debug`, `Clone`, `Serialize`, `Deserialize`, `thiserror::Error` | `@dataclass` generating methods |
| Attribute | `#[x]` on an item | `#[tokio::main]`, `#[pyfunction]`, `#[test]` | A decorator |

```rust
use std::collections::HashMap;

// Your own declarative macro: a dict literal, which Rust does not have
macro_rules! dict {
    ( $( $key:expr => $value:expr ),* $(,)? ) => {{
        let mut map = HashMap::new();
        $( map.insert($key, $value); )*
        map
    }};
}

// What #[derive] saves you from writing
#[derive(Debug, Clone, PartialEq, Default)]
struct Config {
    model: String,
    temperature: f32,
    max_tokens: u32,
}

fn main() {
    let prices = dict! { "idli" => 30, "dosa" => 60, "vada" => 20 };
    println!("{}", prices["dosa"]);

    let base = Config::default();                               // from derive(Default)
    let mine = Config { model: "mini".into(), ..base.clone() }; // from derive(Clone)
    println!("{base:?}");                                       // from derive(Debug)
    println!("{}", mine == base);                               // from derive(PartialEq)

    let x = 7;
    dbg!(x * 2);        // prints file, line, the expression and its value — best quick debug tool
    assert_eq!(x + 1, 8, "maths is broken");
}
```

**Why `println!` must be a macro:** it takes any number of arguments of any types and checks the format string against them at compile time. `println!("{} {}", a)` is a compile error, not a runtime surprise.

**How PyO3, serde and tokio feel like magic:** they are procedural macros. `#[derive(Serialize)]` reads your struct definition and generates the JSON writing code for exactly those fields. `#[pyfunction]` generates the C-ABI wrapper that converts Python arguments to Rust types. Run `cargo expand` to read what they produce — the magic disappears and you see plain Rust.

**Trap:** you will *use* macros daily and *write* them rarely. Do not start a project by writing macros. A function or a generic is almost always clearer.

---

[← 23. `unsafe` and FFI](../part-c-internals/23-unsafe-and-ffi.md) · [Index](../README.md) · [25. Performance checklist →](../part-c-internals/25-performance-checklist.md)
