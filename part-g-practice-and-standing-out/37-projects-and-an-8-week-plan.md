[← 36. Interview Q&A](../part-g-practice-and-standing-out/36-interview-q-a.md) · [Index](../README.md) · [38. One-page cheat sheet →](../part-g-practice-and-standing-out/38-one-page-cheat-sheet.md)

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

[← 36. Interview Q&A](../part-g-practice-and-standing-out/36-interview-q-a.md) · [Index](../README.md) · [38. One-page cheat sheet →](../part-g-practice-and-standing-out/38-one-page-cheat-sheet.md)
