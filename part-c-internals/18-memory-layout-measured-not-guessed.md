[← 17. How rustc turns your code into machine code](../part-c-internals/17-how-rustc-turns-your-code-into-machine-code.md) · [Index](../README.md) · [19. Static vs dynamic dispatch →](../part-c-internals/19-static-vs-dynamic-dispatch.md)

## 18. Memory layout — measured, not guessed

**In one line:** you can ask Rust the exact size of anything, and the answers explain most of its speed.

**Python you know:** `sys.getsizeof` and the `__slots__` trick (your Q40) — and even a bare `int` is 28 bytes.

```rust
use std::mem::size_of;
use std::rc::Rc;
use std::sync::Arc;

// A tiny macro: print the name and size of each type given
macro_rules! show {
    ($($t:ty),* $(,)?) => {
        $( println!("{:<22} {:>3} bytes", stringify!($t), size_of::<$t>()); )*
    };
}

#[allow(dead_code)]
struct Shuffled {       // Rust may reorder fields to waste less space
    a: u8,
    b: u64,
    c: u8,
}

#[allow(dead_code)]
#[repr(C)]
struct COrder {         // #[repr(C)]: keep the written order, like a C struct
    a: u8,
    b: u64,
    c: u8,
}

#[allow(dead_code)]
enum Shape {
    Empty,
    Circle(f32),
    Rect(f32, f32),
}

trait Speak {
    fn hi(&self);
}

fn main() {
    println!("--- plain values");
    show!(i32, f64, char, bool, (), usize);
    println!("--- references and containers");
    show!(&i32, &[f32], &str, String, Vec<f32>, [f32; 4]);
    println!("--- smart pointers");
    show!(Box<i32>, Rc<i32>, Arc<i32>, Box<dyn Speak>, &dyn Speak);
    println!("--- Option");
    show!(Option<i32>, Option<&i32>, Option<Box<i32>>, Option<String>);
    println!("--- structs and enums");
    show!(Shuffled, COrder, Shape);

    // How a Vec grows
    let mut v: Vec<u32> = Vec::new();
    let mut last = usize::MAX;
    for i in 0..40 {
        if v.capacity() != last {
            last = v.capacity();
            print!("len {} cap {} | ", v.len(), last);
        }
        v.push(i);
    }
    println!();

    let exact: Vec<u32> = Vec::with_capacity(1000);
    println!("with_capacity(1000): len {} cap {}", exact.len(), exact.capacity());

    // Stack and heap really are different places
    let on_stack = 42;
    let on_heap = Box::new(42);
    println!("stack {:p}   heap {:p}", &on_stack, on_heap);
}
```

Output on a 64-bit machine (addresses will differ on yours):

```
--- plain values
i32                      4 bytes
f64                      8 bytes
char                     4 bytes
bool                     1 bytes
()                       0 bytes
usize                    8 bytes
--- references and containers
&i32                     8 bytes
&[f32]                  16 bytes
&str                    16 bytes
String                  24 bytes
Vec<f32>                24 bytes
[f32; 4]                16 bytes
--- smart pointers
Box<i32>                 8 bytes
Rc<i32>                  8 bytes
Arc<i32>                 8 bytes
Box<dyn Speak>          16 bytes
&dyn Speak              16 bytes
--- Option
Option<i32>              8 bytes
Option<&i32>             8 bytes
Option<Box<i32>>         8 bytes
Option<String>          24 bytes
--- structs and enums
Shuffled                16 bytes
COrder                  24 bytes
Shape                   12 bytes
len 0 cap 0 | len 1 cap 4 | len 5 cap 8 | len 9 cap 16 | len 17 cap 32 | len 33 cap 64 | 
with_capacity(1000): len 0 cap 1000
stack 0x7ffed7b678dc   heap 0x564ff9d11e20
```

**Read the output — each line is an interview answer**

| Observation | Explanation |
|---|---|
| `&i32` is 8, but `&[f32]` and `&str` are 16 | **Fat pointers.** A slice reference carries pointer + length. |
| `String` and `Vec<f32>` are 24 | Pointer + length + capacity. The elements are on the heap. |
| `[f32; 4]` is 16 | An array is just its elements, inline. No header, no pointer. |
| `Box<dyn Speak>` is 16 | Fat pointer again: pointer to data + pointer to the **vtable** (Section 19). |
| `Option<i32>` is 8 | 4 bytes of data + a tag + padding. |
| `Option<&i32>`, `Option<Box<i32>>` are 8 | **Niche optimisation.** A reference can never be null, so the null bit pattern is used for `None`. A safe nullable pointer for free. |
| `Option<String>` is 24 | Same trick, using an impossible value inside `String`. |
| `Shuffled` 16 vs `COrder` 24 | Rust reorders fields (`b`, then `a`, `c`) to reduce padding. `#[repr(C)]` keeps the order and pays 7 + 7 bytes of padding. You need `#[repr(C)]` only for FFI. |
| `Shape` is 12 | Biggest variant (two `f32` = 8) + a 4-byte tag. |
| Capacity 0 → 4 → 8 → 16 → 32 → 64 | Empty `Vec` allocates nothing. Then it doubles, so pushes are amortised O(1). `with_capacity` skips all the regrowth. |
| `()` is 0 bytes | Zero-sized types exist. A `HashSet<T>` is a `HashMap<T, ()>` and the values take no space. |

**Stack vs heap**

| | Stack | Heap |
|---|---|---|
| What lives there | Local variables whose size is known at compile time | Data whose size is decided at runtime (`Vec`, `String`, `Box`) |
| Allocate / free | Move the stack pointer — practically free | Ask the allocator — much slower |
| Lifetime | Until the function returns | Until the owner is dropped |
| In Python | Almost nothing | Almost everything |

**Why this matters for AI:** an embedding table stored as `Vec<f32>` with one million × 384 values is one continuous 1.5 GB block. The CPU prefetcher streams through it, and SIMD processes 8 or 16 floats per instruction. A `Vec<Vec<f32>>` is one million separate heap blocks — every row is a pointer jump and a likely cache miss. Flat buffers are how ndarray, candle, burn and NumPy all store tensors. You will see this in the wrapper project in Section 26.

**Say this in an interview:** "A `Vec` is pointer-length-capacity on the stack with a contiguous heap buffer. Slices and trait objects are fat pointers. `Option` of a non-nullable pointer uses the null niche, so it is the same size as the pointer. Rust is free to reorder struct fields unless I ask for `repr(C)`."

---

[← 17. How rustc turns your code into machine code](../part-c-internals/17-how-rustc-turns-your-code-into-machine-code.md) · [Index](../README.md) · [19. Static vs dynamic dispatch →](../part-c-internals/19-static-vs-dynamic-dispatch.md)
