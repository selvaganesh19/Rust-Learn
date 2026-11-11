[← 6. Collections — Vec, HashMap, HashSet, tuple](../part-a-the-language-mapped-from-python/06-collections-vec-hashmap-hashset-tuple.md) · [Index](../README.md) · [8. Enums and `match` — Rust's superpower →](../part-a-the-language-mapped-from-python/08-enums-and-match-rust-s-superpower.md)

## 7. Structs and `impl` — the class replacement

**In one line:** data goes in a `struct`, behaviour goes in an `impl` block, and the first parameter tells exactly what the method is allowed to do with the object.

**Python you know:** `class BankAccount` with `__init__` and `self` (your Q12).

```rust
use std::fmt;

#[derive(Debug, Clone, PartialEq)]      // auto-generate __repr__, copy(), __eq__
struct Account {
    owner: String,
    balance: f64,
}

impl Account {
    // No __init__. A constructor is just a normal function that returns Self.
    fn new(owner: &str, balance: f64) -> Self {
        Self { owner: owner.to_string(), balance }
    }

    fn balance(&self) -> f64 {            // &self      -> read only
        self.balance
    }

    fn deposit(&mut self, amount: f64) {  // &mut self  -> may change the object
        self.balance += amount;
    }

    fn close(self) -> f64 {               // self       -> consumes the object
        self.balance                      // after this call the account does not exist
    }
}

impl fmt::Display for Account {           // __str__
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}: ₹{:.2}", self.owner, self.balance)
    }
}

fn transfer(from: &mut Account, to: &mut Account, amount: f64) -> bool {
    if from.balance < amount {
        return false;
    }
    from.balance -= amount;
    to.balance += amount;
    true
}

fn main() {
    let mut selva = Account::new("Selva", 1000.0);
    let mut priya = Account::new("Priya", 500.0);

    selva.deposit(200.0);
    let ok = transfer(&mut selva, &mut priya, 300.0);

    println!("{selva} | {priya} | ok={ok}");      // Display
    println!("{selva:?}");                         // Debug
    println!("{}", selva == selva.clone());        // PartialEq + Clone

    let final_amount = priya.close();
    // println!("{priya}");     // error[E0382]: borrow of moved value: `priya`
    println!("{final_amount} {}", selva.balance());

    // Struct update syntax, tuple struct, unit struct
    let guest = Account { owner: "Guest".into(), ..selva.clone() };
    struct Meters(f64);                            // "newtype": a f64 that cannot be mixed with others
    let height = Meters(1.75);
    println!("{} {}", guest.owner, height.0);
}
```

**`self` in three flavours — this has no Python equivalent**

| Signature | Meaning | Python feeling |
|---|---|---|
| `fn f(&self)` | Borrow, read only | A getter |
| `fn f(&mut self)` | Borrow, may modify. Caller's variable must be `mut`. | A normal method |
| `fn f(self)` | Takes ownership. Object is gone after the call. | No equivalent |
| `fn f() -> Self` | No `self`: associated function, called as `Type::f()` | `@classmethod` / `@staticmethod` |

The `close(self)` trick is powerful: you can make "use after close" a **compile error**. In Python you would write `if self.closed: raise ...` and hope a test catches it.

**Python class feature → Rust**

| Python | Rust |
|---|---|
| `__init__` | `fn new(...) -> Self` (only a convention) |
| `@dataclass` | `#[derive(Debug, Clone, PartialEq)]` |
| `@dataclass(frozen=True)` | Default behaviour — nothing is mutable without `mut` |
| `_private` (convention only) | Fields are private outside the module unless you write `pub` — enforced |
| Class attribute | `const` inside `impl`, or a `static` |
| `@property` | A method `fn name(&self) -> &str` |
| Inheritance | Does not exist. Use composition + traits (Section 10). |

**Inside the machine:** a struct is just its fields laid side by side — no `__dict__`, no object header. `Account` is 32 bytes: 24 for the `String` and 8 for the `f64`. Methods are ordinary functions; `selva.deposit(200.0)` compiles to `Account::deposit(&mut selva, 200.0)` — a direct call, no attribute lookup. This is like your `__slots__` discussion (Q40), except it is always on.

**Trap — no inheritance:** do not try to rebuild `class Dog(Animal)`. Put shared behaviour in a trait, shared data in a field. "Composition over inheritance" is not advice in Rust, it is the only option.

---

[← 6. Collections — Vec, HashMap, HashSet, tuple](../part-a-the-language-mapped-from-python/06-collections-vec-hashmap-hashset-tuple.md) · [Index](../README.md) · [8. Enums and `match` — Rust's superpower →](../part-a-the-language-mapped-from-python/08-enums-and-match-rust-s-superpower.md)
