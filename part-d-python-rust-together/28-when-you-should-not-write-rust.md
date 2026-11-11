[← 27. Wrapper patterns you must know](../part-d-python-rust-together/27-wrapper-patterns-you-must-know.md) · [Index](../README.md) · [29. AI crate map →](../part-e-ai-in-rust/29-ai-crate-map.md)

## 28. When you should NOT write Rust

**In one line:** Rust is a tool for the hot path and the long-running service, not a religion.

| Situation | Better choice | Why |
|---|---|---|
| The work is already one vectorised NumPy / PyTorch call | Stay in Python | That call is already C/C++/CUDA. Rust cannot beat `torch.matmul` on a GPU. |
| Model training, research, notebooks | Python | Ecosystem, iteration speed |
| I/O-bound glue: call an API, write to a DB | Python (asyncio) | The network is the bottleneck, not the interpreter |
| A script that runs once | Python | Compile time and strictness buy nothing |
| The team cannot review Rust | Python, or a small well-tested Rust module | Unmaintainable code is slow code |
| You have not profiled | Profile first (`cProfile`, `py-spy`) | You may be optimising the wrong 5% |

**Where Rust clearly wins:** Python-level loops over millions of items; parsing and tokenising text; custom data structures (indexes, tries, graphs); anything that must use many cores inside one process; low-latency inference servers; CLIs and tools that must start instantly; edge devices and WebAssembly.

**Say this in an interview:** "I profile first. If the hot spot is a Python loop or needs real parallelism, I move that function to Rust behind PyO3. If it is already a vectorised library call, Rust will not help and I leave it."

---

[← 27. Wrapper patterns you must know](../part-d-python-rust-together/27-wrapper-patterns-you-must-know.md) · [Index](../README.md) · [29. AI crate map →](../part-e-ai-in-rust/29-ai-crate-map.md)
