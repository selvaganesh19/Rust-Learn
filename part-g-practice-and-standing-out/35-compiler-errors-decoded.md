[← 34. Your Python problems, solved in Rust](../part-g-practice-and-standing-out/34-your-python-problems-solved-in-rust.md) · [Index](../README.md) · [36. Interview Q&A →](../part-g-practice-and-standing-out/36-interview-q-a.md)

## 35. Compiler errors decoded

Rust errors have a code. `rustc --explain E0382` prints a full explanation with examples. These are the ones you will meet in the first month.

| Code | Message (short) | What it really means | Usual fix |
|---|---|---|---|
| E0382 | use of moved value | You gave the value away and then used the old name | Pass `&x` instead, or `.clone()`, or restructure so the move is last |
| E0499 | cannot borrow as mutable more than once | Two `&mut` to the same thing are alive together | Finish using the first before creating the second; split into smaller scopes |
| E0502 | cannot borrow as mutable because also borrowed as immutable | A reader is still alive while you try to write | Use the reader's result first (copy the value out), then mutate |
| E0505 | cannot move out because it is borrowed | You tried to give away something a reference still points to | Drop or stop using the reference first |
| E0506 | cannot assign because it is borrowed | Same as above, for assignment | Same |
| E0507 | cannot move out of borrowed content / behind a reference | You have `&T` but tried to take the `T` out | `.clone()`, or `std::mem::take`, or `Option::take()`, or match on a reference |
| E0597 | does not live long enough | A reference outlives the value it points to | Move the owner to an outer scope, or return an owned value |
| E0106 | missing lifetime specifier | A returned or stored reference with no way to infer its source | Add `<'a>`, or return an owned type |
| E0373 | closure may outlive the current function | A thread or task closure borrows a local | Add `move` before the closure |
| E0277 | trait bound is not satisfied | The type lacks a capability (`Display`, `Send`, `Clone`, `Serialize`…) | Add `#[derive(...)]`, implement the trait, or add a bound `T: Trait` |
| E0308 | mismatched types | Expected one type, found another | Read "expected … found …". Often a stray `;`, a missing `&`, or `Some(...)` / `Ok(...)` forgotten |
| E0599 | no method named … found | The method does not exist for this type, or its trait is not imported | `use` the trait (e.g. `use rayon::prelude::*`), or check the type is what you think |
| E0282 | type annotations needed | `collect()` / `parse()` / `into()` cannot guess the target type | `let v: Vec<_> = ...` or `.collect::<Vec<_>>()` |
| E0384 | cannot assign twice to immutable variable | Missing `mut` | `let mut x` |
| E0596 | cannot borrow as mutable | The variable or reference is not `mut` | `let mut x`, or take `&mut` in the signature |
| E0425 / E0433 | cannot find value / failed to resolve | Name not in scope | Add a `use`, check spelling, check `mod` declaration, check `Cargo.toml` |
| E0616 | field is private | Used a private field from another module | Add `pub`, or use a method |
| E0004 | non-exhaustive patterns | A `match` misses a case | Add the missing arm |

**How to read a Rust error** (they look long; the structure is always the same):

```
error[E0502]: cannot borrow `scores` as mutable because it is also borrowed as immutable
 --> src/main.rs:5:5
  |
4 |     let first = &scores[0];
  |                  ------ immutable borrow occurs here          <- 1. where the first borrow started
5 |     scores.push(100);
  |     ^^^^^^^^^^^^^^^^ mutable borrow occurs here               <- 2. the line that conflicts
6 |     println!("{first}");
  |               ------- immutable borrow later used here        <- 3. why the first borrow is still alive
```

Three arrows, three facts. Fix any one of them and the error goes: stop using `first` later, or push before borrowing, or copy the value out (`let first = scores[0];`). Read the `help:` line at the bottom too — the compiler often prints the exact fix.

---

[← 34. Your Python problems, solved in Rust](../part-g-practice-and-standing-out/34-your-python-problems-solved-in-rust.md) · [Index](../README.md) · [36. Interview Q&A →](../part-g-practice-and-standing-out/36-interview-q-a.md)
