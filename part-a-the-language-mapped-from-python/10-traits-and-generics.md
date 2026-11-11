[← 9. `Option` and `Result` — no None bugs, no exceptions](../part-a-the-language-mapped-from-python/09-option-and-result-no-none-bugs-no-exceptions.md) · [Index](../README.md) · [11. Closures and iterators →](../part-a-the-language-mapped-from-python/11-closures-and-iterators.md)

## 10. Traits and generics

**In one line:** a trait is a named set of methods a type promises to have — like `Protocol`/`ABC` — and generics let you write one function that works for every type that keeps the promise, with zero runtime cost.

**Python you know:** duck typing ("if it has `.charge()`, call it", your Q18), `ABC`, `Protocol`, and dunder methods.

```rust
use std::fmt::Display;
use std::ops::Add;

// ── The contract ────────────────────────────────────────────────────
trait Embedder {
    fn dim(&self) -> usize;
    fn embed(&self, text: &str) -> Vec<f32>;

    // Default method: every implementor gets it free, and may override it
    fn embed_batch(&self, texts: &[&str]) -> Vec<Vec<f32>> {
        texts.iter().map(|t| self.embed(t)).collect()
    }
}

// ── Two unrelated types keep the promise ────────────────────────────
struct LengthEmbedder;
struct ConstantEmbedder {
    value: f32,
    dim: usize,
}

impl Embedder for LengthEmbedder {
    fn dim(&self) -> usize {
        2
    }
    fn embed(&self, text: &str) -> Vec<f32> {
        vec![text.len() as f32, text.split_whitespace().count() as f32]
    }
}

impl Embedder for ConstantEmbedder {
    fn dim(&self) -> usize {
        self.dim
    }
    fn embed(&self, _text: &str) -> Vec<f32> {
        vec![self.value; self.dim]
    }
}

// ── Static dispatch: the compiler writes one copy of this per type ──
fn show<E: Embedder>(model: &E, text: &str) {
    println!("{:?}", model.embed(text));
}

// Same thing, shorter spelling
fn show_batch(model: &impl Embedder, texts: &[&str]) {
    println!("{:?}", model.embed_batch(texts));
}

// ── Dynamic dispatch: decided while running, like Python ────────────
fn run_all(models: &[Box<dyn Embedder>], text: &str) {
    for m in models {
        println!("dim {} -> {:?}", m.dim(), m.embed(text));
    }
}

// ── Generic function with trait bounds ──────────────────────────────
fn largest<T: PartialOrd + Copy>(items: &[T]) -> Option<T> {
    let mut iter = items.iter().copied();
    let mut best = iter.next()?;          // empty slice -> return None
    for x in iter {
        if x > best {
            best = x;
        }
    }
    Some(best)
}

fn tag<T>(x: T) -> String
where
    T: Display,                           // `where` clause: same bounds, easier to read
{
    format!("<{x}>")
}

// ── Operator overloading = implementing a std trait ─────────────────
#[derive(Debug, Clone, Copy, PartialEq)]
struct Vec2 {
    x: f32,
    y: f32,
}

impl Add for Vec2 {                       // __add__
    type Output = Vec2;
    fn add(self, other: Vec2) -> Vec2 {
        Vec2 { x: self.x + other.x, y: self.y + other.y }
    }
}

impl From<(f32, f32)> for Vec2 {          // conversion: Vec2::from((1.0, 2.0)) or .into()
    fn from(t: (f32, f32)) -> Self {
        Vec2 { x: t.0, y: t.1 }
    }
}

fn main() {
    show(&LengthEmbedder, "rust is fast");
    show_batch(&ConstantEmbedder { value: 0.5, dim: 3 }, &["a", "b"]);

    let models: Vec<Box<dyn Embedder>> = vec![
        Box::new(LengthEmbedder),
        Box::new(ConstantEmbedder { value: 1.0, dim: 2 }),
    ];
    run_all(&models, "hello world");

    println!("{:?} {:?} {:?}", largest(&[3, 9, 4]), largest(&[1.5, 0.2]), largest::<i32>(&[]));
    println!("{} {}", tag(42), tag("hi"));

    let a = Vec2 { x: 1.0, y: 2.0 };
    let b: Vec2 = (0.5, 0.5).into();
    println!("{:?}", a + b);
}
```

**Your dunder methods → Rust traits**

| Python dunder | Rust trait | How you usually get it |
|---|---|---|
| `__repr__` | `Debug` | `#[derive(Debug)]` |
| `__str__` | `Display` | Write `impl fmt::Display` |
| `__eq__` | `PartialEq`, `Eq` | `#[derive(PartialEq, Eq)]` |
| `__hash__` | `Hash` | `#[derive(Hash)]` |
| `__lt__` etc. | `PartialOrd`, `Ord` | `#[derive(PartialOrd, Ord)]` |
| `__add__`, `__mul__` | `Add`, `Mul` (in `std::ops`) | Write `impl Add` |
| `__getitem__` | `Index`, `IndexMut` | Write `impl Index` |
| `__iter__` / `__next__` | `IntoIterator` / `Iterator` | Write `impl Iterator` |
| `__call__` | `Fn`, `FnMut`, `FnOnce` | Closures implement these |
| `__del__`, `__exit__` | `Drop` | Write `impl Drop` |
| `copy.copy` / `deepcopy` | `Copy` / `Clone` | `#[derive(Clone, Copy)]` |
| `__init__` with no args | `Default` | `#[derive(Default)]` |
| `int(x)`, `str(x)` conversions | `From` / `Into`, `FromStr` | Write `impl From<A> for B` |
| `__bool__`, `__len__` | No trait | Just methods `is_empty()`, `len()` |

**Static vs dynamic — pick one consciously**

| | `fn f<T: Trait>(x: &T)` / `impl Trait` | `fn f(x: &dyn Trait)` |
|---|---|---|
| Decided | At compile time | While running |
| Speed | Direct call, can be inlined | One pointer jump through a vtable |
| Binary size | One copy per type used | One copy |
| Mixed types in one `Vec` | Not possible | Yes: `Vec<Box<dyn Trait>>` |
| Use it for | Hot loops, numeric code | Plugins, lists of different things |

**Inside the machine:** generics are compiled by **monomorphisation** — `largest::<i32>` and `largest::<f64>` become two separate machine-code functions, each as fast as if you had written it by hand. That is the meaning of "zero-cost abstraction". Details and proof in Section 19.

**Trap — "trait is not implemented":** `error[E0277]: the trait bound ... is not satisfied` is the Rust form of Python's `AttributeError`, but at compile time. Read which trait is missing, then add a `#[derive(...)]` or a bound like `T: Clone`.

**Say this in an interview:** "Traits give me duck typing with a compile-time contract. With generics the compiler monomorphises — static dispatch, inlinable. With `dyn Trait` I get dynamic dispatch through a vtable when I need mixed types. Rust has no inheritance; I share behaviour with traits and data with composition."

---

[← 9. `Option` and `Result` — no None bugs, no exceptions](../part-a-the-language-mapped-from-python/09-option-and-result-no-none-bugs-no-exceptions.md) · [Index](../README.md) · [11. Closures and iterators →](../part-a-the-language-mapped-from-python/11-closures-and-iterators.md)
