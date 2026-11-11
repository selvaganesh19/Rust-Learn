[Index](../README.md) · [1. Setup and Cargo →](../part-a-the-language-mapped-from-python/01-setup-and-cargo.md)

## 0. Why Rust for AI — the honest picture

**In one line:** Python is the steering wheel of AI, Rust is slowly becoming the engine under it.

You will not train GPT in Rust. Research and training stay in Python + PyTorch. Rust wins in the places where Python is slow or unsafe: data loading, tokenising, serving, vector search, tooling.

You are already running Rust every day without knowing it:

| You type in Python | What actually runs |
|---|---|
| `from tokenizers import Tokenizer` (Hugging Face) | Rust core, Python wrapper (PyO3) |
| `import polars as pl` | Rust DataFrame engine |
| `from pydantic import BaseModel` (v2) | `pydantic-core`, written in Rust |
| `import orjson` | Rust JSON parser |
| `safetensors` model files | Rust library |
| `uv pip install`, `ruff check` | Both tools are Rust binaries |
| Qdrant, LanceDB, Meilisearch | Databases written in Rust |

See the pattern: **Python API on top, Rust core below.** That exact skill — writing a Rust core and wrapping it for Python — is Part D of this file, and very few Python developers have it.

**Where Rust fits in an AI system**

```mermaid
flowchart LR
    A[Raw data] -->|polars, arrow| B[Clean data]
    B -->|tokenizers| C[Token IDs]
    C -->|PyTorch in Python| D[Trained model]
    D -->|safetensors / ONNX| E[Model file]
    E -->|candle, ort, burn| F[Inference in Rust]
    F -->|axum, tokio| G[API server]
    G -->|qdrant, lancedb| H[Vector search / RAG]
```

Only the training box in the middle is firmly Python. Everything left and right of it is where Rust is used in production.

**Why Rust and not C++ for this job**

| Problem | Python | C++ | Rust |
|---|---|---|---|
| Speed | Slow interpreter | Fast | Fast (same LLVM backend as Clang) |
| Use all CPU cores | GIL blocks threads in the standard build | Yes, but data races are your problem | Yes, and data races are compile errors |
| Memory bugs (use-after-free, double free) | Not possible | Common, found at 2 a.m. in production | Rejected by the compiler in safe code |
| `None` crash | `AttributeError` at runtime | Null pointer crash | `Option<T>` forces you to handle it |
| Packaging | pip, venv, wheels | CMake pain | `cargo` — one tool |

**Say this in an interview:** "I use Python for experiments and Rust for the hot path. Rust gives C-level speed with memory safety checked at compile time and no GIL, so one process can use every core. Most fast Python AI libraries — tokenizers, polars, pydantic-core — already follow this design."

---

[Index](../README.md) · [1. Setup and Cargo →](../part-a-the-language-mapped-from-python/01-setup-and-cargo.md)
