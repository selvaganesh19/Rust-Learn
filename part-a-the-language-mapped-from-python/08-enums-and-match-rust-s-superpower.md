[← 7. Structs and `impl` — the class replacement](../part-a-the-language-mapped-from-python/07-structs-and-impl-the-class-replacement.md) · [Index](../README.md) · [9. `Option` and `Result` — no None bugs, no exceptions →](../part-a-the-language-mapped-from-python/09-option-and-result-no-none-bugs-no-exceptions.md)

## 8. Enums and `match` — Rust's superpower

**In one line:** a Rust enum is "this value is exactly one of these shapes, and each shape can carry its own data" — and `match` forces you to handle every shape.

**Python you know:** `Enum` is only names. For "one of several shapes" you used dicts with a `"type"` key, or `match/case` on dicts (your Q50) — but nothing stops you from forgetting a case.

```rust
#[derive(Debug)]
enum Payment {
    Cash,                                   // no data
    Upi { id: String },                     // named fields
    Card { last4: String, emi_months: u8 }, // different fields
}

fn fee(p: &Payment, amount: f64) -> f64 {
    match p {
        Payment::Cash | Payment::Upi { .. } => 0.0,
        Payment::Card { emi_months: 0, .. } => amount * 0.02,
        Payment::Card { emi_months, .. } => amount * 0.02 + f64::from(*emi_months) * 10.0,
    }
    // Add a new variant `Wallet` later -> this function stops compiling
    // until you handle it. The compiler finds every place you forgot.
}

fn describe(n: i32) -> String {
    match n {
        0 => "zero".to_string(),
        x if x < 0 => format!("negative {x}"),        // guard
        1 | 2 | 3 => "small".to_string(),             // or-pattern
        big @ 100.. => format!("big number {big}"),   // bind with @
        _ => "medium".to_string(),
    }
}

fn main() {
    let payments = vec![
        Payment::Cash,
        Payment::Upi { id: "selva@okaxis".into() },
        Payment::Card { last4: "1234".into(), emi_months: 0 },
        Payment::Card { last4: "9876".into(), emi_months: 6 },
    ];
    for p in &payments {
        println!("{p:?} -> fee ₹{}", fee(p, 1000.0));
    }

    // if let: care about only one variant
    if let Payment::Upi { id } = &payments[1] {
        println!("upi id = {id}");
    }

    // let-else: take the value or leave early
    let Payment::Card { last4, .. } = &payments[2] else {
        panic!("expected a card");
    };
    println!("card ending {last4}");

    // matches! gives a bool
    let cards = payments.iter().filter(|p| matches!(p, Payment::Card { .. })).count();

    // Destructure tuples and structs
    let point = (3, -3);
    let place = match point {
        (0, 0) => "origin",
        (x, y) if x == -y => "anti-diagonal",
        (_, 0) => "on x axis",
        _ => "somewhere",
    };

    println!("{cards} {place} {} {} {}", describe(-5), describe(2), describe(500));
}
```

**Why AI code loves enums:** real data comes in shapes. A chat message is `System`, `User`, `Assistant` or `ToolCall { name, args }`. A tensor's device is `Cpu`, `Cuda(index)` or `Metal`. A job is `Queued`, `Running { started }`, `Done(result)` or `Failed(error)`. With an enum, "Done but no result" or "Failed with a result" simply cannot be written. People call this **making illegal states unrepresentable**, and it removes a whole family of bugs before the program runs.

**Inside the machine:** an enum is stored as a small tag (which variant) plus enough space for the biggest variant — like a C tagged union, but safe. `match` compiles to a jump table or a few comparisons, never a dictionary lookup. No heap allocation happens unless a variant itself holds a `String` or `Vec`.

**Trap — `_` hides future bugs:** a catch-all `_ =>` arm means the compiler will stay quiet when someone adds a new variant. In code you own, list every variant; keep `_` for open sets like integers.

**Say this in an interview:** "Rust enums are algebraic data types — sum types. Together with exhaustive `match`, the compiler proves I handled every case, so adding a variant gives me a to-do list of compile errors instead of a production bug."

---

[← 7. Structs and `impl` — the class replacement](../part-a-the-language-mapped-from-python/07-structs-and-impl-the-class-replacement.md) · [Index](../README.md) · [9. `Option` and `Result` — no None bugs, no exceptions →](../part-a-the-language-mapped-from-python/09-option-and-result-no-none-bugs-no-exceptions.md)
