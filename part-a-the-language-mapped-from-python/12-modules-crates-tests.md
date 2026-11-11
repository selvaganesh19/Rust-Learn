[← 11. Closures and iterators](../part-a-the-language-mapped-from-python/11-closures-and-iterators.md) · [Index](../README.md) · [13. Ownership →](../part-b-the-core-that-makes-rust-rust/13-ownership.md)

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

---

[← 11. Closures and iterators](../part-a-the-language-mapped-from-python/11-closures-and-iterators.md) · [Index](../README.md) · [13. Ownership →](../part-b-the-core-that-makes-rust-rust/13-ownership.md)
