[← 31. Web crate map](../part-f-web-in-rust/31-web-crate-map.md) · [Index](../README.md) · [33. FastAPI → axum, plus HTTP client and database →](../part-f-web-in-rust/33-fastapi-axum-plus-http-client-and-database.md)

## 32. A complete axum API

**In one line:** a handler is an `async fn`; its *parameters* declare what it needs from the request (path, JSON body, shared state), and axum extracts and validates them before your code runs.

**Python you know:** FastAPI — typed parameters, Pydantic models, dependency injection.

This is a small semantic-search service: add documents, fetch one, search. The embedding is a toy (hash words into 64 buckets) so the file runs with no model download. Swap `embed()` for candle / ort / fastembed and you have a real service.

```bash
cargo new api && cd api
cargo add axum serde_json
cargo add tokio --features full
cargo add serde --features derive
```

```rust
// cargo add axum serde_json; cargo add tokio --features full; cargo add serde --features derive
use std::collections::HashMap;
use std::sync::Arc;

use axum::extract::{Path, State};
use axum::http::StatusCode;
use axum::response::{IntoResponse, Response};
use axum::routing::{get, post};
use axum::{Json, Router};
use serde::{Deserialize, Serialize};
use serde_json::json;
use tokio::sync::RwLock;

// ---------- data shapes (like Pydantic models) ----------

#[derive(Clone, Serialize)]
struct Doc {
    id: u64,
    text: String,
    #[serde(skip)] // do not send the vector back in JSON
    embedding: Vec<f32>,
}

#[derive(Deserialize)]
struct AddDoc {
    text: String,
}

#[derive(Deserialize)]
struct SearchReq {
    query: String,
    #[serde(default = "default_k")]
    k: usize,
}

fn default_k() -> usize {
    3
}

#[derive(Serialize)]
struct Hit {
    id: u64,
    text: String,
    score: f32,
}

// ---------- shared state (like app.state in FastAPI) ----------

#[derive(Clone, Default)]
struct AppState {
    docs: Arc<RwLock<HashMap<u64, Doc>>>,
}

// ---------- errors: one enum, one place that turns it into HTTP ----------

enum ApiError {
    NotFound(u64),
    BadRequest(String),
}

impl IntoResponse for ApiError {
    fn into_response(self) -> Response {
        let (status, msg) = match self {
            ApiError::NotFound(id) => (StatusCode::NOT_FOUND, format!("doc {id} not found")),
            ApiError::BadRequest(m) => (StatusCode::BAD_REQUEST, m),
        };
        (status, Json(json!({ "error": msg }))).into_response()
    }
}

// ---------- toy embedding: hash every word into one of 64 buckets ----------

fn embed(text: &str) -> Vec<f32> {
    let mut v = vec![0.0f32; 64];
    for word in text.split_whitespace() {
        let h = word
            .to_lowercase()
            .bytes()
            .fold(5381u64, |h, b| h.wrapping_mul(33) ^ b as u64);
        v[(h % 64) as usize] += 1.0;
    }
    let norm = v.iter().map(|x| x * x).sum::<f32>().sqrt();
    if norm > 0.0 {
        v.iter_mut().for_each(|x| *x /= norm);
    }
    v
}

// ---------- handlers ----------

async fn health() -> &'static str {
    "ok"
}

async fn add_doc(
    State(state): State<AppState>,
    Json(body): Json<AddDoc>,
) -> Result<(StatusCode, Json<Doc>), ApiError> {
    if body.text.trim().is_empty() {
        return Err(ApiError::BadRequest("text is empty".into()));
    }
    let mut docs = state.docs.write().await;
    let id = docs.len() as u64 + 1;
    let doc = Doc { id, embedding: embed(&body.text), text: body.text };
    docs.insert(id, doc.clone());
    Ok((StatusCode::CREATED, Json(doc)))
}

async fn get_doc(
    State(state): State<AppState>,
    Path(id): Path<u64>,
) -> Result<Json<Doc>, ApiError> {
    let docs = state.docs.read().await;
    docs.get(&id).cloned().map(Json).ok_or(ApiError::NotFound(id))
}

async fn search(State(state): State<AppState>, Json(req): Json<SearchReq>) -> Json<Vec<Hit>> {
    let docs: Vec<Doc> = state.docs.read().await.values().cloned().collect();

    // CPU-heavy work must NOT run on the async threads -> push it to the blocking pool
    let hits = tokio::task::spawn_blocking(move || {
        let q = embed(&req.query);
        let mut hits: Vec<Hit> = docs
            .into_iter()
            .map(|d| Hit {
                score: d.embedding.iter().zip(&q).map(|(a, b)| a * b).sum(),
                id: d.id,
                text: d.text,
            })
            .collect();
        hits.sort_by(|a, b| b.score.total_cmp(&a.score));
        hits.truncate(req.k);
        hits
    })
    .await
    .expect("search task panicked");

    Json(hits)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/health", get(health))
        .route("/docs", post(add_doc))
        .route("/docs/{id}", get(get_doc))
        .route("/search", post(search))
        .with_state(AppState::default());

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("listening on http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}
```

**Run it and talk to it**

```bash
cargo run --release

curl localhost:3000/health
# ok

curl -X POST localhost:3000/docs -H 'content-type: application/json' -d '{"text":"rust is fast and safe"}'
# {"id":1,"text":"rust is fast and safe"}

curl -X POST localhost:3000/docs -H 'content-type: application/json' -d '{"text":"python is easy to write"}'
# {"id":2,"text":"python is easy to write"}

curl localhost:3000/docs/99
# {"error":"doc 99 not found"}                      (HTTP 404)

curl -X POST localhost:3000/search -H 'content-type: application/json' -d '{"query":"is rust safe","k":1}'
# [{"id":1,"text":"rust is fast and safe","score":0.8728716}]

curl -X POST localhost:3000/docs -H 'content-type: application/json' -d '{"txt":1}'
# Failed to deserialize the JSON body into the target type: missing field `text` ...   (HTTP 422)
```

The last call shows validation you did not write: `Json<AddDoc>` rejected the body because the struct has no field `txt`.

**What to notice**

| Line | Idea |
|---|---|
| `#[derive(Deserialize)] struct AddDoc` | A Pydantic model. Wrong JSON never reaches your handler. |
| `State(state): State<AppState>` | Dependency injection. `AppState` is `Clone`, and inside it is an `Arc`, so each request gets a cheap handle to the same data. |
| `Arc<RwLock<HashMap<..>>>` | Shared mutable state across all worker threads: many readers or one writer. |
| `Result<Json<Doc>, ApiError>` | The handler's type lists every way it can answer. |
| `impl IntoResponse for ApiError` | One place decides how errors look on the wire. |
| `spawn_blocking` | CPU work leaves the async threads (Section 22). In a real service the model call goes here. |
| `"/docs/{id}"` | axum 0.8 path syntax. Tutorials for 0.7 and older write `/docs/:id`, which no longer works. |

**Inside the machine:** `Router` is built once at start-up. For each connection tokio spawns a lightweight task; hyper parses the request; axum matches the route and runs the *extractors* — each handler parameter implements `FromRequest` or `FromRequestParts`, and the compiler generates the glue for your exact parameter list. There is no reflection and no runtime inspection of type hints, which is how FastAPI does it.

**Trap — handler does not compile with a huge error:** almost always an extractor is wrong (order matters: the body extractor such as `Json` must be the last parameter), or something in the state is not `Send + Sync`. Add `#[axum::debug_handler]` above the handler (enable axum's `macros` feature) and the message becomes readable.

**Trap — holding a lock across `.await`:** in `add_doc` the write lock is taken and released without awaiting anything in between. Keep it that way. Compute first, lock briefly, store, unlock.

---

[← 31. Web crate map](../part-f-web-in-rust/31-web-crate-map.md) · [Index](../README.md) · [33. FastAPI → axum, plus HTTP client and database →](../part-f-web-in-rust/33-fastapi-axum-plus-http-client-and-database.md)
