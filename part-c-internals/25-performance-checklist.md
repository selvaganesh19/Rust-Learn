[← 24. Macros](../part-c-internals/24-macros.md) · [Index](../README.md) · [26. PyO3 + maturin — a full wrapper project →](../part-d-python-rust-together/26-pyo3-maturin-a-full-wrapper-project.md)

## 25. Performance checklist

**In one line:** Rust makes fast code possible, not automatic — these are the habits that deliver it.

| # | Habit | Why |
|---|---|---|
| 1 | Measure with `cargo run --release` / `cargo bench` | Debug builds can be 10–100x slower |
| 2 | Borrow (`&T`, `&[T]`, `&str`) instead of `.clone()` in hot paths | Each clone of a `Vec`/`String` is an allocation + copy |
| 3 | `Vec::with_capacity(n)`, `String::with_capacity(n)` | Avoids repeated regrowth |
| 4 | One flat `Vec<f32>` plus a `dim`, not `Vec<Vec<f32>>` | Continuous memory, cache friendly, SIMD friendly |
| 5 | `f32` for ML numbers | Half the memory traffic of `f64`; matches model weights |
| 6 | Iterators (`iter().zip()`) rather than `for i in 0..n { a[i] }` | The compiler can drop bounds checks and vectorise |
| 7 | `par_iter()` for independent items | All cores with one word |
| 8 | Reuse buffers: `v.clear()` then refill | `clear` keeps the capacity |
| 9 | Generics in inner loops, `dyn` at the edges | Inlining |
| 10 | `HashMap::with_capacity`, and a faster hasher for integer keys | Default SipHash is safe but not the fastest |
| 11 | Never block inside async code | Keeps tail latency flat |
| 12 | Profile before changing anything | `cargo flamegraph`, `perf`, `samply` |

**Release profile for a production binary**

```toml
[profile.release]
opt-level = 3
lto = "fat"          # optimise across all crates as one unit
codegen-units = 1    # slower compile, better code
panic = "abort"      # smaller binary. Do NOT set this for a PyO3 extension:
                     # PyO3 turns Rust panics into Python exceptions, which needs unwinding.
strip = true         # remove debug symbols from the shipped file
```

```bash
RUSTFLAGS="-C target-cpu=native" cargo build --release   # use every instruction this CPU has (AVX2, AVX-512, NEON)
```

Only use `target-cpu=native` for binaries that run on the machine that built them. A wheel you publish must run on older CPUs too.

**A micro-benchmark you can trust**

```rust
use std::hint::black_box;
use std::time::Instant;

fn dot(a: &[f32], b: &[f32]) -> f32 {
    a.iter().zip(b).map(|(x, y)| x * y).sum()
}

fn main() {
    let a: Vec<f32> = (0..1024).map(|i| i as f32 * 0.001).collect();
    let b: Vec<f32> = (0..1024).map(|i| 1.0 - i as f32 * 0.001).collect();

    let rounds = 200_000;
    let start = Instant::now();
    let mut acc = 0.0;
    for _ in 0..rounds {
        // black_box stops the optimiser from deleting the work or pre-computing the answer
        acc += dot(black_box(&a), black_box(&b));
    }
    let per_call = start.elapsed() / rounds;
    println!("dot of 1024 floats: {per_call:?} per call (checksum {acc})");
}
```

For real benchmarking use the `criterion` crate: it runs warm-ups, many samples, and reports statistically meaningful changes between runs.

**Say this in an interview:** "First I check it is a release build, then I profile. The usual wins are removing clones and allocations from the hot path, flattening nested vectors into contiguous buffers, and parallelising with rayon. Only after that do I look at SIMD or `unsafe`."

---

[← 24. Macros](../part-c-internals/24-macros.md) · [Index](../README.md) · [26. PyO3 + maturin — a full wrapper project →](../part-d-python-rust-together/26-pyo3-maturin-a-full-wrapper-project.md)
