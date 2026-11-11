# Rust for AI Engineers — From Python to the Metal

> Written for one person: you already know Python (your 4 files), now you want Rust for AI, its internals, and Python + Rust together.
> Every topic stands alone: **what it is → the Python you already know → Rust code → what happens inside the machine → the trap → what to say in an interview.**

**Three promises of this file**

1. Every Rust block that has `fn main()` is complete. Copy it into `src/main.rs`, run `cargo run`, it works. No half snippets.
2. Blocks that need a crate start with a `// cargo add ...` line. Run that line first.
3. Code here was compiled and run with `rustc 1.97`, edition 2024, in November 2026 (any newer stable Rust works). The few blocks that were only compile-checked, or not compiled at all, say so right below the block.

---

## Modules

### [Part A — The language, mapped from Python](Part%20A%20-%20The%20language%2C%20mapped%20from%20Python/The%20language%2C%20mapped%20from%20Python.md)

- 0. Why Rust for AI — the honest picture
- 1. Setup and Cargo
- 2. Python → Rust Rosetta table
- 3. Variables, types, numbers
- 4. Strings — `String` vs `&str`
- 5. Functions and control flow
- 6. Collections — Vec, HashMap, HashSet, tuple
- 7. Structs and `impl` — the class replacement
- 8. Enums and `match` — Rust's superpower
- 9. `Option` and `Result` — no None bugs, no exceptions
- 10. Traits and generics
- 11. Closures and iterators
- 12. Modules, crates, tests

### [Part B — The core that makes Rust, Rust](Part%20B%20-%20The%20core%20that%20makes%20Rust%2C%20Rust/The%20core%20that%20makes%20Rust%2C%20Rust.md)

- 13. Ownership
- 14. Borrowing
- 15. Lifetimes
- 16. Smart pointers — Box, Rc, Arc, RefCell, Mutex, Cow

### [Part C — Internals](Part%20C%20-%20Internals/Internals.md)

- 17. How rustc turns your code into machine code
- 18. Memory layout — measured, not guessed
- 19. Static vs dynamic dispatch
- 20. Drop and RAII
- 21. Concurrency without a GIL
- 22. Async — futures are state machines
- 23. `unsafe` and FFI
- 24. Macros
- 25. Performance checklist

### [Part D — Python + Rust together](Part%20D%20-%20Python%20%2B%20Rust%20together/Python%20%2B%20Rust%20together.md)

- 26. PyO3 + maturin — a full wrapper project
- 27. Wrapper patterns you must know
- 28. When you should NOT write Rust

### [Part E — AI in Rust](Part%20E%20-%20AI%20in%20Rust/AI%20in%20Rust.md)

- 29. AI crate map
- 30. AI crates with code

### [Part F — Web in Rust](Part%20F%20-%20Web%20in%20Rust/Web%20in%20Rust.md)

- 31. Web crate map
- 32. A complete axum API
- 33. FastAPI → axum, plus HTTP client and database

### [Part G — Practice and standing out](Part%20G%20-%20Practice%20and%20standing%20out/Practice%20and%20standing%20out.md)

- 34. Your Python problems, solved in Rust
- 35. Compiler errors decoded
- 36. Interview Q&A
- 37. Projects and an 8-week plan
- 38. One-page cheat sheet
