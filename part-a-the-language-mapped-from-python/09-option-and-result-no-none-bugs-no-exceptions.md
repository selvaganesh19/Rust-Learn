[← 8. Enums and `match` — Rust's superpower](../part-a-the-language-mapped-from-python/08-enums-and-match-rust-s-superpower.md) · [Index](../README.md) · [10. Traits and generics →](../part-a-the-language-mapped-from-python/10-traits-and-generics.md)

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

[← 8. Enums and `match` — Rust's superpower](../part-a-the-language-mapped-from-python/08-enums-and-match-rust-s-superpower.md) · [Index](../README.md) · [10. Traits and generics →](../part-a-the-language-mapped-from-python/10-traits-and-generics.md)
