[← 28. When you should NOT write Rust](../part-d-python-rust-together/28-when-you-should-not-write-rust.md) · [Index](../README.md) · [30. AI crates with code →](../part-e-ai-in-rust/30-ai-crates-with-code.md)

## 29. AI crate map

Versions are the latest on crates.io on 11 November 2026. The Rust AI ecosystem changes API between minor versions, so always open the crate's page on docs.rs for the version you install.

| Job | Crate | Version | Python equivalent | Note |
|---|---|---|---|---|
| N-dimensional arrays | `ndarray` | 0.17 | NumPy | CPU, the base of many crates |
| DataFrames | `polars` | 0.55 | pandas / polars | Lazy, parallel, Arrow memory |
| Columnar memory format | `arrow` | 60 | pyarrow | Zero-copy exchange between tools |
| Data parallelism | `rayon` | 1.12 | multiprocessing | `par_iter()` |
| Deep learning — inference first | `candle-core`, `candle-nn`, `candle-transformers` | 0.11 | PyTorch (small) | From Hugging Face; CPU, CUDA, Metal, WASM |
| Deep learning — training + inference | `burn` | 0.22 | PyTorch | Many backends; big API change in 0.22 |
| Run ONNX models | `ort` | 2.0.0-rc.15 | onnxruntime | Wraps Microsoft's ONNX Runtime |
| Bindings to libtorch | `tch` | 0.27 | PyTorch C++ | Needs libtorch installed |
| Tokenisation | `tokenizers` | 0.23 | tokenizers | The same Rust core Python uses |
| Model weight files | `safetensors` | 0.8 | safetensors | Safe, memory-mappable |
| Download from Hugging Face Hub | `hf-hub` | 1.0 | huggingface_hub | |
| Local text embeddings | `fastembed` | 7.7 | fastembed / sentence-transformers | ONNX under the hood |
| Local LLM inference | `mistralrs` | 0.8 | llama-cpp-python / vLLM | Also see `llama-cpp-2` (llama.cpp bindings) |
| Classical ML | `linfa` | 0.8 | scikit-learn | k-means, SVM, linear models… |
| LLM apps, agents, RAG | `rig-core` | 0.44 | LangChain / LlamaIndex | |
| OpenAI-compatible API client | `async-openai` | 0.43 | openai | Works with any compatible server |
| Vector database client | `qdrant-client` | 1.20 | qdrant-client | Qdrant itself is written in Rust |
| Embedded vector database | `lancedb` | 0.41 | lancedb | |
| Full-text search | `tantivy` | 0.27 | Lucene / Whoosh | For hybrid search in RAG |
| Python bindings | `pyo3`, `numpy` | 0.30 | — | Part D |

**How to choose a deep-learning crate**

| You want to… | Use |
|---|---|
| Run a model exported from PyTorch as ONNX, fastest path to production | `ort` |
| Run Hugging Face models (BERT, Whisper, LLMs) in pure Rust, small binary | `candle` |
| Write and **train** models in Rust, same code for training and serving | `burn` |
| Reuse exact PyTorch behaviour and CUDA kernels | `tch` |
| Just get embeddings for RAG | `fastembed` |

---

[← 28. When you should NOT write Rust](../part-d-python-rust-together/28-when-you-should-not-write-rust.md) · [Index](../README.md) · [30. AI crates with code →](../part-e-ai-in-rust/30-ai-crates-with-code.md)
