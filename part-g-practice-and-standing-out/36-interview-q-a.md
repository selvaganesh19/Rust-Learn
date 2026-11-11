[← 35. Compiler errors decoded](../part-g-practice-and-standing-out/35-compiler-errors-decoded.md) · [Index](../README.md) · [37. Projects and an 8-week plan →](../part-g-practice-and-standing-out/37-projects-and-an-8-week-plan.md)

## 36. Interview Q&A

Short answers you can say aloud. The section number tells you where the full explanation is.

**Language core**

**Q1. How is Rust memory-safe without a garbage collector?** (13, 14)
Ownership: each value has one owner and is freed when the owner goes out of scope. Borrowing: references are checked at compile time so they can never outlive the value, and you may have many readers or one writer, never both. All checks happen at compile time, so there is no runtime cost.

**Q2. Move vs Copy vs Clone?** (13)
Move: bitwise copy of the stack part, and the source becomes unusable. Copy: the same bitwise copy, but the source stays valid — only for simple types with no heap data. Clone: an explicit, possibly expensive deep copy through `.clone()`.

**Q3. `String` vs `&str`?** (4)
`String` is an owned, growable, heap-allocated UTF-8 buffer: pointer, length, capacity. `&str` is a borrowed view: pointer and length. Take `&str` in parameters, return `String`.

**Q4. What are lifetimes?** (15)
Compile-time labels for how long references are valid. They do not change runtime behaviour. Mostly inferred; I annotate when a returned reference could come from more than one input or when a struct stores a reference.

**Q5. `Box` vs `Rc` vs `Arc`?** (16)
`Box`: single owner on the heap. `Rc`: shared ownership with a non-atomic count, single thread only. `Arc`: the same with an atomic count, safe across threads.

**Q6. What is interior mutability?** (16)
Mutating through a shared reference, with the borrow rule checked at runtime instead of compile time: `Cell`/`RefCell` for one thread, `Mutex`/`RwLock`/atomics for many.

**Q7. `Option`/`Result` vs exceptions? When do you panic?** (9)
Failure is part of the return type, so callers must handle it; `?` propagates. No hidden control flow. Panic only for bugs — broken invariants — never for expected failures like bad input or a missing file.

**Q8. Trait objects vs generics?** (10, 19)
Generics are monomorphised: static dispatch, inlinable, bigger binary. `dyn Trait` is a fat pointer with a vtable: dynamic dispatch, supports mixed types in one collection, no inlining.

**Q9. What is a zero-cost abstraction? Give an example.** (11, 19)
An abstraction that compiles to the same machine code as the hand-written low-level version. Iterator chains become one loop; `Option<&T>` is the size of a pointer; generics have no dispatch cost.

**Q10. `iter()` vs `into_iter()` vs `iter_mut()`?** (11)
`iter` yields `&T`, `iter_mut` yields `&mut T`, `into_iter` yields `T` and consumes the collection.

**Q11. Can a safe Rust program leak memory?** (16)
Yes. `Rc` cycles, `std::mem::forget`, `Box::leak`. Leaks are considered safe — they are not undefined behaviour. Rust guarantees no use-after-free and no data races, not "no leaks".

**Q12. What does `unsafe` allow, and what does it not?** (23)
Five things: dereference raw pointers, call unsafe functions, access mutable statics, implement unsafe traits, access union fields. It does not turn off the borrow checker or type checker.

**Internals**

**Q13. Walk me through compilation.** (17)
Source → AST → HIR (type check) → MIR (borrow check) → monomorphisation → LLVM IR → optimised machine code → linker. Safety checks are finished before LLVM ever sees the code.

**Q14. Why is `Vec<Vec<f32>>` a poor choice for an embedding matrix?** (18)
Each inner `Vec` is its own heap allocation, so rows are scattered: pointer chasing, cache misses, no SIMD across rows. One flat `Vec<f32>` with a known `dim` is contiguous and is what every tensor library does.

**Q15. How does Rust prevent data races?** (21)
`Send` and `Sync` marker traits plus the borrow rule. To share across threads a type must be `Sync`; to mutate shared data you must go through `Mutex`, `RwLock` or atomics. The compiler rejects anything else.

**Q16. How does async work underneath?** (22)
`async fn` compiles to a state machine implementing `Future`. An executor polls it; at each `.await` that is not ready it returns `Pending` and registers a waker. tokio multiplexes many such tasks over a small thread pool using the OS event queue.

**Q17. Why must inference not run directly in an async handler?** (22, 32)
It is CPU-bound and never yields, so it blocks a runtime worker thread and stalls every other task on it. Use `spawn_blocking` or a dedicated thread pool.

**Python + Rust**

**Q18. How do you call Rust from Python?** (26)
PyO3 for bindings, maturin to build wheels. `#[pyfunction]`, `#[pyclass]`, `#[pymodule]` generate the CPython glue.

**Q19. How do you pass a NumPy array without copying?** (26, 27)
Take `PyReadonlyArray<f32, D>` from the `numpy` crate. It borrows NumPy's buffer; `as_array()` gives an ndarray view, `as_slice()` a `&[f32]` if contiguous.

**Q20. What about the GIL?** (27)
A PyO3 function holds the GIL by default. For long work that does not touch Python objects I wrap it in `py.detach` (called `allow_threads` before PyO3 0.26), which releases the GIL so other Python threads run and my Rust code can use all cores with rayon.

**Q21. What happens if Rust panics inside a Python call?** (27)
PyO3 catches the unwind at the boundary and raises `PanicException` in Python. The interpreter survives. That is also why I do not set `panic = "abort"` for extension modules.

**Q22. When would you not use Rust?** (28)
When the hot path is already a vectorised native call, for training and exploration, for I/O-bound glue, and when nobody else on the team can maintain it. I profile before deciding.

**Q23. Rust vs C++ for ML infrastructure?**
Same performance class and the same LLVM backend. Rust adds compile-time memory and thread safety, a single build tool, and easy Python packaging with maturin. C++ has the larger existing ecosystem — CUDA kernels, most framework internals — so the two often sit side by side through FFI.

---

[← 35. Compiler errors decoded](../part-g-practice-and-standing-out/35-compiler-errors-decoded.md) · [Index](../README.md) · [37. Projects and an 8-week plan →](../part-g-practice-and-standing-out/37-projects-and-an-8-week-plan.md)
