[← 32. A complete axum API](../part-f-web-in-rust/32-a-complete-axum-api.md) · [Index](../README.md) · [34. Your Python problems, solved in Rust →](../part-g-practice-and-standing-out/34-your-python-problems-solved-in-rust.md)

## 33. FastAPI → axum, plus HTTP client and database

| FastAPI | axum |
|---|---|
| `@app.get("/items/{id}")` | `.route("/items/{id}", get(handler))` |
| `def f(id: int)` path parameter | `Path(id): Path<u64>` |
| `def f(q: str = None, limit: int = 10)` query | `Query(p): Query<Params>` where `Params` derives `Deserialize` |
| `def f(body: Item)` Pydantic body | `Json(body): Json<Item>` |
| `Header()` | `HeaderMap`, or `TypedHeader` from `axum-extra` |
| `Depends(get_db)` / `app.state` | `State(state): State<AppState>` |
| `raise HTTPException(404)` | `return Err(ApiError::NotFound(..))` + `IntoResponse` |
| `response_model=Item` | Return type `Json<Item>` |
| `status_code=201` | Return `(StatusCode::CREATED, Json(item))` |
| Middleware | `.layer(...)` with tower / tower-http layers |
| `BackgroundTasks` | `tokio::spawn(async move { ... })` |
| `StreamingResponse` / SSE for LLM tokens | `axum::response::sse::Sse` |
| WebSocket | `axum::extract::ws::WebSocketUpgrade` |
| `/docs` Swagger UI, automatic | Not built in — add `utoipa` |
| `uvicorn main:app --workers 4` | One binary. tokio already uses all cores. |
| `pytest` + `TestClient` | `cargo test` + `tower::ServiceExt::oneshot` |

**HTTP client — calling another service (or an LLM API)**

```rust
// cargo add anyhow serde_json; cargo add reqwest --features json; cargo add tokio --features full; cargo add serde --features derive
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct SearchReq<'a> {
    query: &'a str,
    k: usize,
}

#[derive(Debug, Deserialize)]
struct Hit {
    id: u64,
    text: String,
    score: f32,
}

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = reqwest::Client::new(); // create once, reuse: it holds a connection pool

    let hits: Vec<Hit> = client
        .post("http://localhost:3000/search")
        .json(&SearchReq { query: "is rust safe", k: 2 })
        .send()
        .await?
        .error_for_status()? // 4xx / 5xx -> Err
        .json()
        .await?;

    for h in hits {
        println!("{} {:.3} {}", h.id, h.score, h.text);
    }
    Ok(())
}
```

Compile-checked with reqwest 0.13. Start the server from Section 32 first, then run this. The same pattern calls any OpenAI-compatible chat endpoint: POST a JSON body, add `.bearer_auth(key)`, deserialise the reply into your own struct.

**Database with `sqlx` — the idea (not compiled here, needs a live database)**

```rust
// cargo add sqlx --features runtime-tokio,postgres
// Shape of the code. Check docs.rs/sqlx for the exact API of your version.
#[derive(sqlx::FromRow)]
struct User {
    id: i64,
    name: String,
}

async fn find_user(pool: &sqlx::PgPool, id: i64) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as::<_, User>("SELECT id, name FROM users WHERE id = $1")
        .bind(id)                      // parameters are bound, never string-formatted: no SQL injection
        .fetch_optional(pool)          // Option<User>: "no row" is not an error
        .await
}
```

Create the pool once in `main`, put it inside `AppState`, and every handler gets it through `State`. What makes `sqlx` special: its `query!` macros connect to your database **at compile time** and check that the SQL is valid and that column types match your Rust types. A typo in a column name becomes a compile error.

**Shipping it — a small Docker image**

```dockerfile
FROM rust:1-bookworm AS build
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=build /app/target/release/api /usr/local/bin/api
EXPOSE 3000
CMD ["api"]
```

The final image has no compiler, no interpreter and no `site-packages` — only the binary and a minimal OS. (Build stage and runtime stage use the same Debian release, so the C library versions match.)

**Say this in an interview:** "axum handlers are async functions whose parameters are extractors — `Path`, `Query`, `Json`, `State` — resolved at compile time. Shared state is an `Arc` inside a cloneable struct. Errors implement `IntoResponse`. Middleware is tower layers. Blocking or CPU-heavy work such as inference goes through `spawn_blocking` so the tokio workers never stall."

---

[← 32. A complete axum API](../part-f-web-in-rust/32-a-complete-axum-api.md) · [Index](../README.md) · [34. Your Python problems, solved in Rust →](../part-g-practice-and-standing-out/34-your-python-problems-solved-in-rust.md)
