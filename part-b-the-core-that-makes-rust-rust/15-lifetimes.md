[← 14. Borrowing](../part-b-the-core-that-makes-rust-rust/14-borrowing.md) · [Index](../README.md) · [16. Smart pointers — Box, Rc, Arc, RefCell, Mutex, Cow →](../part-b-the-core-that-makes-rust-rust/16-smart-pointers-box-rc-arc-refcell-mutex-cow.md)

## 15. Lifetimes

**In one line:** a lifetime is the compiler's name for "how long this reference stays valid"; you write `'a` only when the compiler cannot work out by itself which input a returned reference comes from.

**Python you know:** nothing like this. The GC keeps every object alive as long as anything points to it. Rust has no GC, so it must *prove* that no reference outlives the thing it points to.

Lifetimes do not change how long anything lives. They are labels that let the compiler check your claim. Think of a visitor pass that prints "valid only while the host is in the building".

```rust
// Two borrowed inputs, one borrowed output. Which input does the output come from?
// The compiler will not guess. 'a says: "the result is valid only while BOTH inputs are".
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() >= b.len() { a } else { b }
}

// A struct that stores a reference must name the lifetime
#[derive(Debug)]
struct Chunk<'a> {
    text: &'a str,      // points INTO the document: zero copy
    start: usize,
}

fn split_chunks<'a>(doc: &'a str) -> Vec<Chunk<'a>> {
    let base = doc.as_ptr() as usize;
    doc.split(". ")
        .map(|s| Chunk { text: s, start: s.as_ptr() as usize - base })
        .collect()
}

// One reference in, one out: the compiler fills in the lifetime itself ("elision")
fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}

fn main() {
    let doc = String::from("Rust is fast. Python is easy. Use both");
    let chunks = split_chunks(&doc);
    println!("{chunks:?}");
    // drop(doc);              // error[E0505]: cannot move out of `doc` because it is borrowed
    println!("{}", chunks[1].text);

    let a = String::from("embedding");
    let res: String;
    {
        let b = String::from("token");
        res = longest(&a, &b).to_string();   // make an owned copy -> safe to keep after `b` dies
        // If `res` were a `&str` holding `longest(&a, &b)` directly:
        // error[E0597]: `b` does not live long enough
    }
    println!("{res} {}", first_word("hello world"));

    let forever: &'static str = "string literals live for the whole program";
    println!("{forever}");
}
```

**When do you actually write `'a`?**

| Situation | Do you write a lifetime? |
|---|---|
| Function takes references, returns an owned value (`String`, `Vec`, number) | No |
| Function takes **one** reference and returns a reference | No — elision handles it |
| Method with `&self` returning a reference | No — the result is assumed to borrow from `self` |
| Function takes **two or more** references and returns a reference | **Yes** |
| Struct or enum has a reference field | **Yes** |
| `'static` | Means "can live for the whole program": literals, leaked data, or owned data with no borrows |

**Inside the machine:** lifetimes exist only during compilation. They are erased before code generation — the binary contains no trace of them and they cost nothing at runtime. The borrow checker works on MIR (Section 17), computes the region of code where each reference is alive, and rejects the program if a reference could be used after its owner is gone.

**Trap — fighting lifetimes in structs:** when you are new, a struct with `&'a str` fields will spread `<'a>` over your whole program. First make it work with owned fields (`String`, `Vec<T>`). Switch to borrowed fields only where a profiler shows that copying matters — parsers, tokenizers, zero-copy readers.

**Trap — returning a reference to a local:** `fn f() -> &str { let s = String::from("x"); &s }` cannot work in any language without a GC: `s` dies when the function returns. Return the `String` itself.

**Say this in an interview:** "Lifetimes are compile-time annotations that relate the validity of references. They do not extend anything; they let the borrow checker prove no reference outlives its referent. Most are inferred by elision rules, and I annotate when a returned reference could come from more than one input or when a struct holds a borrow."

---

[← 14. Borrowing](../part-b-the-core-that-makes-rust-rust/14-borrowing.md) · [Index](../README.md) · [16. Smart pointers — Box, Rc, Arc, RefCell, Mutex, Cow →](../part-b-the-core-that-makes-rust-rust/16-smart-pointers-box-rc-arc-refcell-mutex-cow.md)
