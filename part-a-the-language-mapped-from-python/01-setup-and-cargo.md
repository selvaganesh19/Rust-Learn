[← 0. Why Rust for AI — the honest picture](../part-a-the-language-mapped-from-python/00-why-rust-for-ai-the-honest-picture.md) · [Index](../README.md) · [2. Python → Rust Rosetta table →](../part-a-the-language-mapped-from-python/02-python-rust-rosetta-table.md)

## 1. Setup and Cargo

**In one line:** `cargo` is pip + venv + pytest + black + build tool in one command.

```bash
# install (Linux / macOS). On Windows download rustup-init.exe from rustup.rs
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

rustc --version          # compiler
cargo --version          # build tool + package manager
rustup update stable     # upgrade Rust (a new stable version comes every 6 weeks)

cargo new hello          # create project (binary)
cd hello
cargo run                # compile + run (debug build, slow code, fast compile)
cargo run --release      # optimised build — ALWAYS use this when you measure speed
```

**Project layout**

```
hello/
├── Cargo.toml        # like pyproject.toml — name, version, dependencies
├── Cargo.lock        # like poetry.lock / uv.lock — exact versions
├── src/
│   └── main.rs       # entry point: fn main()
└── target/           # build output (like __pycache__, never commit it)
```

**Daily commands, mapped from Python**

| You do in Python | You do in Rust |
|---|---|
| `python main.py` | `cargo run` |
| `pip install requests` | `cargo add reqwest` |
| `pip install "uvicorn[standard]"` (extras) | `cargo add tokio --features full` (features) |
| `python -m venv .venv` | Not needed. Every project has its own dependencies. |
| `pytest` | `cargo test` |
| `black .` | `cargo fmt` |
| `ruff check` / `pylint` | `cargo clippy` |
| `mypy .` | `cargo check` (type check only, very fast) |
| `python -m pydoc` | `cargo doc --open` |
| PyPI | crates.io (docs for every crate at docs.rs) |

**Editor:** VS Code + the `rust-analyzer` extension. It shows types and errors as you type. Do not learn Rust without it.

**Trap:** beginners benchmark `cargo run` (debug build) and say "Rust is slow". Debug builds have zero optimisation and can be 10–100x slower. Measure only with `--release`.

---

[← 0. Why Rust for AI — the honest picture](../part-a-the-language-mapped-from-python/00-why-rust-for-ai-the-honest-picture.md) · [Index](../README.md) · [2. Python → Rust Rosetta table →](../part-a-the-language-mapped-from-python/02-python-rust-rosetta-table.md)
