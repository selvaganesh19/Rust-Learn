[← 3. Variables, types, numbers](../part-a-the-language-mapped-from-python/03-variables-types-numbers.md) · [Index](../README.md) · [5. Functions and control flow →](../part-a-the-language-mapped-from-python/05-functions-and-control-flow.md)

## 4. Strings — `String` vs `&str`

**In one line:** `String` owns its text on the heap; `&str` is a borrowed window (pointer + length) into text owned by someone else.

**Python you know:** one `str` type, immutable, indexing gives a character.

Think of a hostel room. `String` is the student who has the room — can move furniture, can vacate it. `&str` is a visitor's pass — can look, cannot change, and is valid only while the room exists.

```rust
fn main() {
    let literal: &str = "hello";                    // text baked into the program binary
    let mut owned: String = String::from("hello");  // heap buffer, growable
    owned.push_str(" world");
    owned.push('!');

    let view: &str = &owned[0..5];                  // slice: pointer + length, nothing copied
    println!("{view} | {owned} | {}", literal.len());

    // Converting
    let s1 = "abc".to_string();        // &str -> String (allocates)
    let s2: String = "abc".into();     // same thing
    let s3 = format!("{s1}-{s2}-{}", 42);   // f-string that returns a String
    let r: &str = s3.as_str();         // String -> &str (free)
    println!("{r}");

    // UTF-8: len() is BYTES, not characters
    let tamil = "வணக்கம்";
    println!("bytes={} chars={}", tamil.len(), tamil.chars().count());   // bytes=21 chars=7
    // let c = tamil[0];               // error[E0277]: `str` cannot be indexed by `{integer}`
    println!("{:?}", tamil.chars().next());          // Some('வ')

    // The methods you used in Python
    let s = "  Hello World Python  ";
    println!("{}", s.trim().to_uppercase());                     // .strip().upper()
    println!("{:?}", s.split_whitespace().collect::<Vec<_>>());  // .split()
    println!("{}", s.trim().replace("Hello", "Hi"));             // .replace()
    println!("{} {:?}", s.contains("World"), s.find("World"));   // `in`, .find()
    println!("{}", s.trim().chars().rev().collect::<String>());  // s[::-1]
    println!("{}", ["a", "b", "c"].join(","));                   // ",".join(...)
    println!("{:?} {}", "42".parse::<i32>(), "4x".parse::<i32>().is_err());   // int("42")

    // Formatting — almost the same mini-language as f-strings
    let (name, bal) = ("Selva", 12345.678);
    println!("{name:>10}|{bal:.2}|{:05}|{:#x}|{:?}", 42, 255, "dbg");
}
```

**Which one to use**

| Situation | Use | Why |
|---|---|---|
| Function parameter that only reads text | `&str` | Accepts both `&String` and literals, no copy |
| Struct field that must keep the text | `String` | The struct owns it, no lifetime headache |
| Returning newly built text | `String` | Caller becomes the owner |
| Raw bytes (file, network) | `Vec<u8>` / `&[u8]` | Not guaranteed to be UTF-8 |
| File paths | `PathBuf` / `&Path` | Paths are not always valid UTF-8 |

**Inside the machine**

```
let owned = String::from("hello");

 STACK (24 bytes)                 HEAP
┌──────────┐                    ┌───┬───┬───┬───┬───┐
│ ptr    ──┼──────────────────► │ h │ e │ l │ l │ o │
│ len = 5  │                    └───┴───┴───┴───┴───┘
│ cap = 5  │                      ▲
└──────────┘                      │
                                  │
let view: &str = &owned[1..4];    │   (16 bytes on stack)
┌──────────┐                      │
│ ptr    ──┼────────────── points at 'e'
│ len = 3  │
└──────────┘
```

`String` is three machine words: pointer, length, capacity. `&str` is two: pointer, length. A slice costs nothing — this is why Rust parsers (tokenizers, JSON) are fast: they cut the input into thousands of `&str` pieces without copying one byte.

**Trap — indexing:** `s[0]` does not compile, because one "character" can be 1 to 4 bytes in UTF-8, so `s[0]` cannot be O(1). Use `s.chars().nth(0)` for characters, `s.as_bytes()[0]` for bytes. Slicing `&s[0..2]` uses byte positions and **panics** if you cut a character in the middle.

**Trap — `+` in a loop:** same lesson as your Python file (Q6). `s = s + "x"` works but prefer `s.push_str("x")`, or `String::with_capacity(n)` when you know the size.

**Say this in an interview:** "`String` is an owned, growable UTF-8 buffer with pointer, length and capacity. `&str` is a fat pointer — pointer and length — into UTF-8 bytes it does not own. APIs should take `&str` and return `String`."

---

[← 3. Variables, types, numbers](../part-a-the-language-mapped-from-python/03-variables-types-numbers.md) · [Index](../README.md) · [5. Functions and control flow →](../part-a-the-language-mapped-from-python/05-functions-and-control-flow.md)
