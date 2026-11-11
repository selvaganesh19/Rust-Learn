[← 19. Static vs dynamic dispatch](../part-c-internals/19-static-vs-dynamic-dispatch.md) · [Index](../README.md) · [21. Concurrency without a GIL →](../part-c-internals/21-concurrency-without-a-gil.md)

## 20. Drop and RAII

**In one line:** cleanup code runs automatically, at a known point — the moment the owner goes out of scope — on every path out of the function, including early returns and errors.

**Python you know:** `with open(...) as f:` and `try/finally`, because `__del__` runs "whenever the GC feels like it" (your Q13).

```rust
use std::time::Instant;

struct Timer {
    label: &'static str,
    start: Instant,
}

impl Timer {
    fn new(label: &'static str) -> Self {
        println!("[{label}] start");
        Timer { label, start: Instant::now() }
    }
}

impl Drop for Timer {                    // __exit__ and __del__ in one, but with exact timing
    fn drop(&mut self) {
        println!("[{}] finished in {:?}", self.label, self.start.elapsed());
    }
}

struct Conn(&'static str);

impl Drop for Conn {
    fn drop(&mut self) {
        println!("closing connection {}", self.0);
    }
}

fn query(fail: bool) -> Result<u32, String> {
    let _c = Conn("db");
    if fail {
        return Err("query failed".into());   // `_c` is dropped here too. No finally needed.
    }
    Ok(42)
}

fn main() {
    let _whole = Timer::new("main");

    {
        let _t = Timer::new("load model");
        let _a = Conn("a");
        let _b = Conn("b");
        println!("working...");
    }   // dropped in REVERSE order of creation: b, a, then the timer

    println!("{:?}", query(true));
    println!("{:?}", query(false));

    let early = Conn("early");
    drop(early);                             // free it now instead of at the end of scope
    println!("end of main");
}   // `_whole` dropped here
```

**Where you rely on this without noticing**

| Resource | Type | What `drop` does |
|---|---|---|
| Heap memory | `Vec`, `String`, `Box` | Frees the buffer |
| File | `std::fs::File` | Closes the file descriptor |
| Lock | `MutexGuard` | Unlocks the mutex |
| Shared ownership | `Rc`, `Arc` | Decrements the count; frees at zero |
| Network socket | `TcpStream` | Closes the connection |
| Database transaction | e.g. `sqlx::Transaction` | Rolls back if you did not commit |
| GPU tensor | framework tensor types | Releases device memory |

**Inside the machine:** the compiler works out, on MIR, every point where a value's scope ends and inserts a call to `drop` there — it is an ordinary function call placed at compile time. No finaliser queue, no GC pause. For AI serving this matters: latency does not have random spikes from garbage collection, and a large tensor's memory is returned the instant the request handler finishes.

**Trap — `let _ = value;` drops immediately:** a bare `_` does not bind anything, so the value is dropped on that line. `let _guard = mutex.lock();` keeps the lock until the end of scope; `let _ = mutex.lock();` unlocks at once. One underscore character, very different programs.

**Trap — holding a lock too long:** a `MutexGuard` lives until the end of its scope. Wrap the locked part in its own `{ }` block, or call `drop(guard)`, before doing slow work.

---

[← 19. Static vs dynamic dispatch](../part-c-internals/19-static-vs-dynamic-dispatch.md) · [Index](../README.md) · [21. Concurrency without a GIL →](../part-c-internals/21-concurrency-without-a-gil.md)
