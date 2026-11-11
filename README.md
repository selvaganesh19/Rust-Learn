# Rust for AI Engineers — From Python to the Metal

> Written for one person: you already know Python (your 4 files), now you want Rust for AI, its internals, and Python + Rust together.
> Every topic stands alone: **what it is → the Python you already know → Rust code → what happens inside the machine → the trap → what to say in an interview.**

**Three promises of this file**

1. Every Rust block that has `fn main()` is complete. Copy it into `src/main.rs`, run `cargo run`, it works. No half snippets.
2. Blocks that need a crate start with a `// cargo add ...` line. Run that line first.
3. Code here was compiled and run with `rustc 1.97`, edition 2024, in November 2026 (any newer stable Rust works). The few blocks that were only compile-checked, or not compiled at all, say so right below the block.

---

## Table of Contents

**Part A — The language, mapped from Python**

0. [Why Rust for AI — the honest picture](part-a-the-language-mapped-from-python/00-why-rust-for-ai-the-honest-picture.md)
1. [Setup and Cargo](part-a-the-language-mapped-from-python/01-setup-and-cargo.md)
2. [Python → Rust Rosetta table](part-a-the-language-mapped-from-python/02-python-rust-rosetta-table.md)
3. [Variables, types, numbers](part-a-the-language-mapped-from-python/03-variables-types-numbers.md)
4. [Strings — `String` vs `&str`](part-a-the-language-mapped-from-python/04-strings-string-vs-str.md)
5. [Functions and control flow](part-a-the-language-mapped-from-python/05-functions-and-control-flow.md)
6. [Collections — Vec, HashMap, HashSet, tuple](part-a-the-language-mapped-from-python/06-collections-vec-hashmap-hashset-tuple.md)
7. [Structs and `impl` — the class replacement](part-a-the-language-mapped-from-python/07-structs-and-impl-the-class-replacement.md)
8. [Enums and `match` — Rust's superpower](part-a-the-language-mapped-from-python/08-enums-and-match-rust-s-superpower.md)
9. [`Option` and `Result` — no None bugs, no exceptions](part-a-the-language-mapped-from-python/09-option-and-result-no-none-bugs-no-exceptions.md)
10. [Traits and generics](part-a-the-language-mapped-from-python/10-traits-and-generics.md)
11. [Closures and iterators](part-a-the-language-mapped-from-python/11-closures-and-iterators.md)
12. [Modules, crates, tests](part-a-the-language-mapped-from-python/12-modules-crates-tests.md)

**Part B — The core that makes Rust, Rust**

13. [Ownership](part-b-the-core-that-makes-rust-rust/13-ownership.md)
14. [Borrowing](part-b-the-core-that-makes-rust-rust/14-borrowing.md)
15. [Lifetimes](part-b-the-core-that-makes-rust-rust/15-lifetimes.md)
16. [Smart pointers — Box, Rc, Arc, RefCell, Mutex, Cow](part-b-the-core-that-makes-rust-rust/16-smart-pointers-box-rc-arc-refcell-mutex-cow.md)

**Part C — Internals**

17. [How rustc turns your code into machine code](part-c-internals/17-how-rustc-turns-your-code-into-machine-code.md)
18. [Memory layout — measured, not guessed](part-c-internals/18-memory-layout-measured-not-guessed.md)
19. [Static vs dynamic dispatch](part-c-internals/19-static-vs-dynamic-dispatch.md)
20. [Drop and RAII](part-c-internals/20-drop-and-raii.md)
21. [Concurrency without a GIL](part-c-internals/21-concurrency-without-a-gil.md)
22. [Async — futures are state machines](part-c-internals/22-async-futures-are-state-machines.md)
23. [`unsafe` and FFI](part-c-internals/23-unsafe-and-ffi.md)
24. [Macros](part-c-internals/24-macros.md)
25. [Performance checklist](part-c-internals/25-performance-checklist.md)

**Part D — Python + Rust together**

26. [PyO3 + maturin — a full wrapper project](part-d-python-rust-together/26-pyo3-maturin-a-full-wrapper-project.md)
27. [Wrapper patterns you must know](part-d-python-rust-together/27-wrapper-patterns-you-must-know.md)
28. [When you should NOT write Rust](part-d-python-rust-together/28-when-you-should-not-write-rust.md)

**Part E — AI in Rust**

29. [AI crate map](part-e-ai-in-rust/29-ai-crate-map.md)
30. [AI crates with code](part-e-ai-in-rust/30-ai-crates-with-code.md)

**Part F — Web in Rust**

31. [Web crate map](part-f-web-in-rust/31-web-crate-map.md)
32. [A complete axum API](part-f-web-in-rust/32-a-complete-axum-api.md)
33. [FastAPI → axum, plus HTTP client and database](part-f-web-in-rust/33-fastapi-axum-plus-http-client-and-database.md)

**Part G — Practice and standing out**

34. [Your Python problems, solved in Rust](part-g-practice-and-standing-out/34-your-python-problems-solved-in-rust.md)
35. [Compiler errors decoded](part-g-practice-and-standing-out/35-compiler-errors-decoded.md)
36. [Interview Q&A](part-g-practice-and-standing-out/36-interview-q-a.md)
37. [Projects and an 8-week plan](part-g-practice-and-standing-out/37-projects-and-an-8-week-plan.md)
38. [One-page cheat sheet](part-g-practice-and-standing-out/38-one-page-cheat-sheet.md)
