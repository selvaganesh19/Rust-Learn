[← 21. Concurrency without a GIL](../part-c-internals/21-concurrency-without-a-gil.md) · [Index](../README.md) · [23. `unsafe` and FFI →](../part-c-internals/23-unsafe-and-ffi.md)

## 22. Async — futures are state machines

**In one line:** `async fn` returns a *future* — a value describing work that has not started yet; a runtime (tokio) polls many futures on a few threads, so one process can wait on thousands of network calls at once.

**Python you know:** `async def`, `await`, `asyncio.gather`, `asyncio.Semaphore` (your Q42).

When to use what: **waiting** (HTTP calls, database, LLM APIs) → async. **Computing** (tokenising, matrix maths) → threads / rayon. The same rule as in Python, except Rust threads actually work for computing.

```rust
// cargo add tokio --features full
use std::sync::Arc;
use std::time::{Duration, Instant};
use tokio::sync::Semaphore;
use tokio::task::JoinSet;

async fn fake_llm_call(id: u32) -> String {
    tokio::time::sleep(Duration::from_millis(300)).await;   // stands in for a network call
    format!("answer {id}")
}

#[tokio::main]                      // starts the runtime, then runs this async main
async fn main() {
    let start = Instant::now();

    // A fixed number of calls at the same time: asyncio.gather(a, b)
    let (a, b) = tokio::join!(fake_llm_call(1), fake_llm_call(2));
    println!("{a}, {b} after {:?}", start.elapsed());        // ~300 ms, not 600

    // Many calls, at most 5 in flight: gather + Semaphore
    let limit = Arc::new(Semaphore::new(5));
    let mut tasks = JoinSet::new();
    for id in 0..20 {
        let limit = Arc::clone(&limit);
        tasks.spawn(async move {
            let _permit = limit.acquire_owned().await.unwrap();   // released when dropped
            fake_llm_call(id).await
        });
    }
    let mut answers = Vec::new();
    while let Some(result) = tasks.join_next().await {
        answers.push(result.unwrap());
    }
    println!("{} answers after {:?}", answers.len(), start.elapsed());   // 20 calls / 5 at a time

    // Timeout: asyncio.wait_for
    let slow = tokio::time::timeout(Duration::from_millis(100), fake_llm_call(99)).await;
    println!("timed out: {}", slow.is_err());

    // CPU-heavy work must leave the async threads: loop.run_in_executor / asyncio.to_thread
    let total = tokio::task::spawn_blocking(|| (1..=5_000_000u64).map(|x| x % 7).sum::<u64>())
        .await
        .unwrap();
    println!("blocking result {total}");
}
```

**asyncio → tokio**

| asyncio | tokio |
|---|---|
| `asyncio.run(main())` | `#[tokio::main] async fn main()` |
| `await coro()` | `fut().await` |
| `asyncio.gather(a(), b())` | `tokio::join!(a(), b())` |
| `asyncio.create_task(c())` | `tokio::spawn(c())` |
| `asyncio.TaskGroup` | `tokio::task::JoinSet` |
| `asyncio.sleep(1)` | `tokio::time::sleep(Duration::from_secs(1)).await` |
| `asyncio.wait_for(c(), 5)` | `tokio::time::timeout(dur, c()).await` |
| `asyncio.Semaphore(5)` | `tokio::sync::Semaphore::new(5)` |
| `asyncio.Queue` | `tokio::sync::mpsc::channel` |
| `asyncio.Lock` | `tokio::sync::Mutex` |
| `asyncio.to_thread(f)` | `tokio::task::spawn_blocking(f)` |
| `aiohttp` / `httpx` | `reqwest` |

**Inside the machine**

- The compiler rewrites each `async fn` into an anonymous `enum` — a state machine. Every `.await` is a state. Local variables that must survive across an `.await` become fields of that enum.
- That enum implements the `Future` trait, which has one method: `poll`. Polling runs the code up to the next point where it would wait, then returns `Pending`; or finishes and returns `Ready(value)`.
- Nothing runs until something polls it. Calling `fake_llm_call(1)` without `.await` does **nothing** (and the compiler warns you).
- The runtime is a library, not part of the language. tokio keeps a pool of worker threads and a queue of tasks. When a socket has data, the OS (epoll / kqueue / IOCP) tells tokio, which wakes exactly the task that was waiting.
- A task has no stack of its own; it is just that state-machine value, often a few hundred bytes. That is how one server holds hundreds of thousands of open connections.

**The big difference from asyncio:** asyncio runs every task on one thread. tokio's default runtime runs tasks on **all cores**. So a spawned task must be `Send`, and shared state needs `Arc` — the compiler checks both.

**Trap — blocking the runtime:** a long CPU loop or `std::thread::sleep` inside an async function freezes one of tokio's worker threads, and every task queued on it waits. Model inference inside a handler is exactly this mistake. Move it to `spawn_blocking` or a rayon pool. (Same rule as "never call `time.sleep` in asyncio", but more costly to get wrong.)

**Trap — `std::sync::Mutex` across `.await`:** holding a normal mutex guard while awaiting can deadlock or fail to compile (the guard is not `Send`). Either release the lock before the `.await`, or use `tokio::sync::Mutex`.

**Say this in an interview:** "An `async fn` compiles to a state machine implementing `Future`. Futures are lazy and are driven by an executor calling `poll`; a waker tells the executor when to poll again. tokio is a multi-threaded work-stealing executor on top of the OS event queue. I keep CPU-bound work off it with `spawn_blocking`."

---

[← 21. Concurrency without a GIL](../part-c-internals/21-concurrency-without-a-gil.md) · [Index](../README.md) · [23. `unsafe` and FFI →](../part-c-internals/23-unsafe-and-ffi.md)
