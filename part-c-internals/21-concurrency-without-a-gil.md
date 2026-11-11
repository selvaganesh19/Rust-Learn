[← 20. Drop and RAII](../part-c-internals/20-drop-and-raii.md) · [Index](../README.md) · [22. Async — futures are state machines →](../part-c-internals/22-async-futures-are-state-machines.md)

## 21. Concurrency without a GIL

**In one line:** Rust threads really run in parallel on all cores, and the compiler refuses to build a program that has a data race.

**Python you know:** in the standard build the GIL lets one thread run Python bytecode at a time, so CPU work needs `multiprocessing`, with pickling and separate memory (your Q41). Python 3.13+ also offers an optional free-threaded build, but the default interpreter still has the GIL.

Two marker traits make "fearless concurrency" work. The compiler derives them automatically:

| Trait | Meaning | Is | Is not |
|---|---|---|---|
| `Send` | Safe to **move** to another thread | Almost everything, `Arc<T>`, `Mutex<T>` | `Rc<T>` (non-atomic counter) |
| `Sync` | Safe to **share by reference** between threads | `i32`, `&str`, `Arc<T>`, `Mutex<T>` | `RefCell<T>`, `Cell<T>` |

`thread::spawn` requires its closure to be `Send`. Try to send an `Rc` and you get a compile error — not a crash once a month in production.

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::mpsc;
use std::thread;

fn is_prime(n: u64) -> bool {
    if n < 2 {
        return false;
    }
    let mut i = 2;
    while i * i <= n {
        if n % i == 0 {
            return false;
        }
        i += 1;
    }
    true
}

fn main() {
    let limit = 200_000u64;
    let workers = 4u64;

    // ── Scoped threads: may borrow local variables, joined automatically ──
    let hits = AtomicUsize::new(0);
    thread::scope(|s| {
        for w in 0..workers {
            let hits = &hits;                          // plain borrow, no Arc needed
            s.spawn(move || {
                // worker w checks w, w+4, w+8, ...
                let n = (w..limit).step_by(workers as usize).filter(|&n| is_prime(n)).count();
                hits.fetch_add(n, Ordering::Relaxed);  // lock-free counter
            });
        }
    });
    println!("primes below {limit}: {}", hits.load(Ordering::Relaxed));

    // ── Channels: send messages instead of sharing memory (like queue.Queue) ──
    let (tx, rx) = mpsc::channel();
    for id in 0..3 {
        let tx = tx.clone();
        thread::spawn(move || {
            let answer = id * id;
            tx.send((id, answer)).unwrap();
        });
    }
    drop(tx);                                          // close the channel so the loop below ends
    let mut results: Vec<(i32, i32)> = rx.iter().collect();
    results.sort();
    println!("{results:?}");

    // ── This is the bug Rust will not compile ──
    // let shared = std::rc::Rc::new(5);
    // thread::spawn(move || println!("{shared}"));
    // error[E0277]: `Rc<i32>` cannot be sent between threads safely
}
```

**Data parallelism with `rayon` — the crate you will use most in AI data code.** Change `.iter()` to `.par_iter()` and the work spreads over every core.

```rust
// cargo add rayon
use rayon::prelude::*;
use std::time::Instant;

fn main() {
    let data: Vec<f64> = (0..20_000_000).map(|x| x as f64).collect();

    let t = Instant::now();
    let seq: f64 = data.iter().map(|x| x.sqrt().sin()).sum();
    let t_seq = t.elapsed();

    let t = Instant::now();
    let par: f64 = data.par_iter().map(|x| x.sqrt().sin()).sum();
    let t_par = t.elapsed();

    println!("sequential {seq:.3} in {t_seq:?}");
    println!("parallel   {par:.3} in {t_par:?}  ({} threads)", rayon::current_num_threads());

    // Typical uses: tokenise documents, parse files, score vectors
    let docs = vec!["rust is fast", "python is easy", "use both together"];
    let lengths: Vec<usize> = docs.par_iter().map(|d| d.split_whitespace().count()).collect();
    println!("{lengths:?}");
}
```

Run it with `cargo run --release`. The speed-up is close to the number of physical cores for CPU-bound work like this. (The two sums can differ in the last decimal places: floating-point addition is done in a different order.)

**Python tool → Rust tool**

| Python | Rust | Note |
|---|---|---|
| `threading.Thread` | `std::thread::spawn` / `thread::scope` | Truly parallel |
| `multiprocessing.Pool.map` | `rayon`'s `par_iter().map()` | Same process, shared memory, no pickling |
| `queue.Queue` | `std::sync::mpsc`, or the `crossbeam` crate | |
| `threading.Lock` | `Mutex<T>` | The lock wraps the data |
| A counter protected by a lock | `AtomicUsize` | Lock-free |
| `asyncio` | `tokio` | Next section |

**Inside the machine:** `std::thread` threads are real OS threads (1:1). `rayon` keeps a pool of threads, one per core, and uses *work stealing*: `par_iter` splits the data in halves recursively, and an idle thread steals a half from a busy one. Atomics compile to single CPU instructions (`lock add` on x86). None of this needs a runtime that pauses the world.

**Trap — `Mutex` poisoning:** if a thread panics while holding a lock, later `.lock()` calls return `Err`. That is why you see `.lock().unwrap()` everywhere. It is deliberate: the data may be half-updated.

**Trap — deadlock is still possible:** Rust prevents data races, not logic errors. Two threads taking two locks in opposite order will still freeze. Always take locks in the same order.

**Say this in an interview:** "`Send` means a value can move to another thread, `Sync` means `&T` can be shared. They are auto-derived and checked at compile time, so a data race is a type error. For CPU-bound data work I use rayon's work-stealing pool; for shared state, `Arc<Mutex<T>>` or atomics; for pipelines, channels."

---

[← 20. Drop and RAII](../part-c-internals/20-drop-and-raii.md) · [Index](../README.md) · [22. Async — futures are state machines →](../part-c-internals/22-async-futures-are-state-machines.md)
