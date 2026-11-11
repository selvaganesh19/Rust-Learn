[← 22. Async — futures are state machines](../part-c-internals/22-async-futures-are-state-machines.md) · [Index](../README.md) · [24. Macros →](../part-c-internals/24-macros.md)

## 23. `unsafe` and FFI

**In one line:** `unsafe` does not switch the checks off — it unlocks five extra abilities, and inside that block *you* promise what the compiler can no longer prove.

**Python you know:** `ctypes` — you can call C, and if you pass the wrong pointer the interpreter dies.

The five things `unsafe` allows:

1. Dereference a raw pointer (`*const T`, `*mut T`)
2. Call an `unsafe` function, including any C function
3. Read or write a mutable `static`
4. Implement an `unsafe` trait (such as `Send` by hand)
5. Access a field of a `union`

Everything else — the borrow checker, type checking — still runs inside the block.

```rust
use std::ffi::c_char;

// Declare a function from the C standard library (edition 2024 syntax)
unsafe extern "C" {
    fn strlen(s: *const c_char) -> usize;
}

// The usual pattern: a small unsafe core behind a safe function
fn sum_every_second(data: &[f32]) -> f32 {
    let mut total = 0.0;
    let mut i = 0;
    while i < data.len() {
        // SAFETY: the loop condition guarantees i < data.len()
        total += unsafe { *data.get_unchecked(i) };   // index with no bounds check
        i += 2;
    }
    total
}

fn main() {
    let v = vec![10, 20, 30];
    let p: *const i32 = v.as_ptr();            // making a raw pointer is safe
    let second = unsafe { *p.add(1) };         // using it is not
    println!("{second}");

    let text = c"hello from C";                // C string literal: ends with a \0 byte
    let n = unsafe { strlen(text.as_ptr()) };
    println!("{n}");

    println!("{}", sum_every_second(&[1.0, 2.0, 3.0, 4.0, 5.0]));
}
```

**Calling Rust from Python the raw way (no PyO3)** — good to understand once, because PyO3 does this for you underneath.

```rust
// src/lib.rs      Cargo.toml needs:  [lib]  crate-type = ["cdylib"]
// build with:     cargo build --release   ->  target/release/libfastdot.so

/// # Safety
/// `a` and `b` must each point to `n` valid f32 values.
#[unsafe(no_mangle)]                           // keep the symbol name `dot` in the .so file
pub unsafe extern "C" fn dot(a: *const f32, b: *const f32, n: usize) -> f32 {
    let a = unsafe { std::slice::from_raw_parts(a, n) };
    let b = unsafe { std::slice::from_raw_parts(b, n) };
    a.iter().zip(b).map(|(x, y)| x * y).sum()
}
```

```python
import ctypes
import numpy as np

lib = ctypes.CDLL("./target/release/libfastdot.so")     # .dylib on macOS, .dll on Windows
f32p = ctypes.POINTER(ctypes.c_float)
lib.dot.argtypes = [f32p, f32p, ctypes.c_size_t]
lib.dot.restype = ctypes.c_float

a = np.arange(4, dtype=np.float32)        # [0, 1, 2, 3]
b = np.ones(4, dtype=np.float32)
print(lib.dot(a.ctypes.data_as(f32p), b.ctypes.data_as(f32p), len(a)))   # 6.0
```

It works, but you manage every type by hand, and one wrong `argtypes` entry gives a segmentation fault. PyO3 (Section 26) gives you the same speed with real Python types, exceptions and classes.

**Inside the machine:** `extern "C"` selects the platform's C calling convention — which registers hold which arguments — so any language that can call C can call your function. `#[unsafe(no_mangle)]` stops rustc from renaming the symbol. `cdylib` produces a normal shared library. This is the door through which Rust enters Python, Node, Java, Go and everything else.

**Where real projects use `unsafe`:** FFI (Python, C, CUDA), SIMD intrinsics, custom allocators, and the insides of data structures like `Vec` and `HashMap`. Application code and web handlers almost never need it.

**Trap — `unsafe` to silence the borrow checker:** if the compiler rejects your safe code, the code is usually wrong. Converting references to raw pointers to get around it brings back exactly the bugs Rust exists to prevent — and now they are your responsibility.

**Say this in an interview:** "`unsafe` marks code where I uphold invariants the compiler cannot check — raw pointers, FFI, and so on. The discipline is a small unsafe core with a `// SAFETY:` comment explaining why it holds, wrapped in a safe API. Safe code calling it cannot cause undefined behaviour if that core is right."

---

[← 22. Async — futures are state machines](../part-c-internals/22-async-futures-are-state-machines.md) · [Index](../README.md) · [24. Macros →](../part-c-internals/24-macros.md)
