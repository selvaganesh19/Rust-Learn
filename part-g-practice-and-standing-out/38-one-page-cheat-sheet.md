[← 37. Projects and an 8-week plan](../part-g-practice-and-standing-out/37-projects-and-an-8-week-plan.md) · [Index](../README.md)

## 38. One-page cheat sheet

```rust
// ── Variables ────────────────────────────────────────────────────────
let x = 5;                      // immutable
let mut y = 5;                  // mutable
const N: usize = 10;            // constant
let x = x + 1;                  // shadowing

// ── Types ────────────────────────────────────────────────────────────
// i8..i128  u8..u128  isize usize  f32 f64  bool  char  ()  &str String
// [T; N] array   &[T] slice   (A, B) tuple   Vec<T>   HashMap<K, V>   HashSet<T>
// Option<T> = Some(T) | None           Result<T, E> = Ok(T) | Err(E)

// ── Ownership ────────────────────────────────────────────────────────
let a = String::from("x");
let b = a;                      // move: `a` is gone
let c = b.clone();              // deep copy
let r = &c;                     // shared borrow   (many allowed)
let m = &mut y;                 // exclusive borrow (only one, no readers)

// ── Functions ────────────────────────────────────────────────────────
fn read(v: &[i32]) -> i32 { v.iter().sum() }          // borrow
fn change(v: &mut Vec<i32>) { v.push(1); }            // mutable borrow
fn take(v: Vec<i32>) -> usize { v.len() }             // takes ownership
fn pick<'a>(a: &'a str, b: &'a str) -> &'a str { a }  // lifetime

// ── Struct, enum, trait ──────────────────────────────────────────────
#[derive(Debug, Clone, PartialEq)]
struct Point { x: f32, y: f32 }
impl Point {
    fn new(x: f32, y: f32) -> Self { Self { x, y } }
    fn len(&self) -> f32 { (self.x * self.x + self.y * self.y).sqrt() }
}
enum Shape { Circle(f32), Rect { w: f32, h: f32 } }
trait Area { fn area(&self) -> f32; }
impl Area for Shape {
    fn area(&self) -> f32 {
        match self {
            Shape::Circle(r) => 3.14159 * r * r,
            Shape::Rect { w, h } => w * h,
        }
    }
}

// ── Control flow ─────────────────────────────────────────────────────
if let Some(v) = maybe { }                    // one pattern
let Some(v) = maybe else { return; };         // or leave
for (i, item) in list.iter().enumerate() { }
while let Some(top) = stack.pop() { }
let n = loop { break 42; };

// ── Errors ───────────────────────────────────────────────────────────
let n: i32 = text.parse()?;                           // propagate
let n = text.parse::<i32>().unwrap_or(0);             // default
let v = map.get("k").ok_or("missing key")?;           // Option -> Result
let r = thing().map_err(|e| format!("failed: {e}"))?; // convert error

// ── Iterators ────────────────────────────────────────────────────────
v.iter().map(|x| x * 2).filter(|x| *x > 3).collect::<Vec<_>>();
v.iter().sum::<i32>();   v.iter().max();   v.iter().any(|&x| x > 0);
v.iter().zip(&w);        v.chunks(8);      v.windows(2);
v.sort();   v.sort_by(|a, b| b.cmp(a));   v.sort_by_key(|p| p.age);   v.dedup();   v.retain(|x| *x > 0);

// ── Smart pointers and threads ───────────────────────────────────────
Box::new(x)                 // heap, one owner
Rc::new(x)                  // shared, one thread
Arc::new(Mutex::new(x))     // shared + mutable, many threads
thread::spawn(move || { })  // new OS thread
data.par_iter().map(f)      // rayon: all cores

// ── Async ────────────────────────────────────────────────────────────
#[tokio::main] async fn main() { let (a, b) = tokio::join!(f(), g()); }
tokio::spawn(async move { });                 // background task
tokio::task::spawn_blocking(|| heavy());      // CPU work off the runtime

// ── PyO3 ─────────────────────────────────────────────────────────────
#[pyfunction] fn f(py: Python<'_>, a: Vec<f32>) -> PyResult<f32> { Ok(py.detach(|| work(&a))) }
#[pyclass] struct Index { }      #[pymethods] impl Index { #[new] fn new() -> Self { } }
#[pymodule] fn mymod(m: &Bound<'_, PyModule>) -> PyResult<()> { m.add_function(wrap_pyfunction!(f, m)?) }
```

(The cheat sheet is a reference card, not one runnable program.)

**Commands**

```bash
cargo new app            cargo add serde --features derive      cargo check
cargo run --release      cargo test                             cargo clippy
cargo fmt                cargo doc --open                       cargo build --release
maturin develop --release                                       rustc --explain E0382
```

**The ten sentences to remember**

1. One owner per value; when the owner dies, the value is freed.
2. Many readers or one writer — never both.
3. A reference can never outlive what it points to; lifetimes are how the compiler checks that.
4. `String` owns, `&str` borrows. `Vec<T>` owns, `&[T]` borrows.
5. No `null`: `Option`. No exceptions: `Result` and `?`.
6. `enum` + `match` = every case handled, checked by the compiler.
7. Traits are the contract; generics are free at runtime, `dyn` costs one jump.
8. No GIL: `Send` and `Sync` make data races a compile error.
9. Async is for waiting, threads are for computing; never block the runtime.
10. Python on top, Rust underneath: PyO3, NumPy without copies, release the GIL.

---

*Tested with rustc 1.97 (edition 2024), PyO3 0.30, axum 0.8, tokio 1.54, candle 0.11, burn 0.22, polars 0.55 — November 2026. Stack this with your `PYTHON_MASTERY.md`: Python for the idea, Rust for the engine.*

---

[← 37. Projects and an 8-week plan](../part-g-practice-and-standing-out/37-projects-and-an-8-week-plan.md) · [Index](../README.md)
