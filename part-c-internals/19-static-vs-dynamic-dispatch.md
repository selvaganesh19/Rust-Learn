[← 18. Memory layout — measured, not guessed](../part-c-internals/18-memory-layout-measured-not-guessed.md) · [Index](../README.md) · [20. Drop and RAII →](../part-c-internals/20-drop-and-raii.md)

## 19. Static vs dynamic dispatch

**In one line:** generics are resolved at compile time into separate specialised functions (fast, bigger binary); `dyn Trait` is resolved at runtime through a table of function pointers (flexible, one extra jump).

**Python you know:** every method call is dynamic — look up the attribute on the object, then on its class, then along the MRO, every single time.

```rust
trait Activation {
    fn apply(&self, x: f32) -> f32;
}

struct Relu;
struct Scale(f32);

impl Activation for Relu {
    fn apply(&self, x: f32) -> f32 {
        x.max(0.0)
    }
}

impl Activation for Scale {
    fn apply(&self, x: f32) -> f32 {
        x * self.0
    }
}

// Static dispatch. The compiler generates forward_static::<Relu> and forward_static::<Scale>.
fn forward_static<A: Activation>(act: &A, xs: &[f32]) -> Vec<f32> {
    xs.iter().map(|&x| act.apply(x)).collect()
}

// Dynamic dispatch. One function; each call goes through the vtable.
fn forward_dyn(act: &dyn Activation, xs: &[f32]) -> Vec<f32> {
    xs.iter().map(|&x| act.apply(x)).collect()
}

fn main() {
    let xs = [-1.0, 0.5, 2.0];

    println!("{:?}", forward_static(&Relu, &xs));
    println!("{:?}", forward_static(&Scale(10.0), &xs));

    // Only dyn can hold different types in one collection
    let layers: Vec<Box<dyn Activation>> = vec![Box::new(Scale(2.0)), Box::new(Relu)];
    let mut out = xs.to_vec();
    for layer in &layers {
        out = forward_dyn(layer.as_ref(), &out);
    }
    println!("{out:?}");
}
```

**What a `dyn` pointer really is**

```
Box<dyn Activation>   (16 bytes)
┌────────────┐          ┌──────────────────────┐
│ data ptr ──┼────────► │ the Scale value (2.0)│
│ vtable ptr─┼───┐      └──────────────────────┘
└────────────┘   │      vtable for (Scale as Activation) — one per type, in read-only memory
                 └────► ┌──────────────────────┐
                        │ drop function        │
                        │ size, alignment      │
                        │ fn apply ────────────┼──► machine code of Scale::apply
                        └──────────────────────┘
```

A dynamic call = load the function address from the vtable, then call it. That costs a couple of nanoseconds, but more importantly the optimiser cannot inline through it, so it cannot vectorise the loop around it.

**Monomorphisation in one picture**

```
You write:          fn forward_static<A: Activation>(act: &A, xs: &[f32])

Compiler makes:     fn forward_static_Relu(act: &Relu, xs: &[f32])    // apply() inlined: x.max(0.0)
                    fn forward_static_Scale(act: &Scale, xs: &[f32])  // apply() inlined: x * factor
```

Each copy is as fast as hand-written code. This is the "zero-cost abstraction" promise: what you do not use, you do not pay for; what you use, you could not write faster by hand. The price is paid in compile time and binary size.

**Rule of thumb for AI code:** per-element or per-token work → generics. Per-request or per-layer choices (which model, which backend, which plugin) → `dyn` is fine, the call happens rarely.

**Trap — not every trait can be `dyn`:** a trait with generic methods, or methods that return `Self`, is not "dyn compatible" (older docs say "object safe"), because a vtable cannot hold an unknown number of versions. The compiler tells you; the fix is usually `where Self: Sized` on that method or a different design.

**Say this in an interview:** "Generics are monomorphised — static dispatch, inlinable, no runtime cost, larger binary. `dyn Trait` is a fat pointer to data plus vtable — dynamic dispatch, one indirect call, no inlining. I use generics in hot paths and trait objects at architectural boundaries."

---

[← 18. Memory layout — measured, not guessed](../part-c-internals/18-memory-layout-measured-not-guessed.md) · [Index](../README.md) · [20. Drop and RAII →](../part-c-internals/20-drop-and-raii.md)
