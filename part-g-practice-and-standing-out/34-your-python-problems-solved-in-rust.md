[← 33. FastAPI → axum, plus HTTP client and database](../part-f-web-in-rust/33-fastapi-axum-plus-http-client-and-database.md) · [Index](../README.md) · [35. Compiler errors decoded →](../part-g-practice-and-standing-out/35-compiler-errors-decoded.md)

## 34. Your Python problems, solved in Rust

The same problems from your `PYTHON_BASIC_CODING.md`. Every function is checked by an `assert` in `main`, so if this file runs silently to the last line, every answer is right.

```rust
use std::collections::{HashMap, HashSet};
use std::hash::Hash;

// Q1 — palindrome number (maths, no string)
fn is_palindrome_num(n: i64) -> bool {
    if n < 0 {
        return false;
    }
    let (mut x, mut rev) = (n, 0);
    while x > 0 {
        rev = rev * 10 + x % 10;
        x /= 10;
    }
    rev == n
}

// Q2 — Armstrong number
fn is_armstrong(n: u32) -> bool {
    let digits = n.to_string();
    let power = digits.len() as u32;
    digits.chars().map(|c| c.to_digit(10).unwrap().pow(power)).sum::<u32>() == n
}

// Q3 — prime. `all` on an empty range is true, so 2 and 3 work.
fn is_prime(n: u64) -> bool {
    n >= 2 && (2..).take_while(|i| i * i <= n).all(|i| n % i != 0)
}

// Q4 — Fibonacci
fn fibonacci(n: usize) -> Vec<u64> {
    let mut out = Vec::with_capacity(n);
    let (mut a, mut b) = (0u64, 1u64);
    for _ in 0..n {
        out.push(a);
        (a, b) = (b, a + b);      // tuple assignment, like Python
    }
    out
}

// Q5 — factorial. Python ints never overflow; here overflow becomes None.
fn factorial(n: u32) -> Option<u64> {
    (1..=u64::from(n)).try_fold(1u64, |acc, x| acc.checked_mul(x))
}

// Q8 — sum of digits
fn digit_sum(n: i64) -> u32 {
    n.unsigned_abs().to_string().chars().filter_map(|c| c.to_digit(10)).sum()
}

// Q12 — palindrome string (ignore case, spaces, punctuation)
fn is_palindrome_str(s: &str) -> bool {
    let clean: Vec<char> = s.chars().filter(|c| c.is_alphanumeric()).flat_map(char::to_lowercase).collect();
    clean.iter().eq(clean.iter().rev())
}

// Q13 — vowels and consonants
fn count_vowels_consonants(s: &str) -> (usize, usize) {
    let (mut vowels, mut consonants) = (0, 0);
    for ch in s.to_lowercase().chars() {
        if "aeiou".contains(ch) {
            vowels += 1;
        } else if ch.is_alphabetic() {
            consonants += 1;
        }
    }
    (vowels, consonants)
}

// Q14 — reverse word order
fn reverse_words(s: &str) -> String {
    s.split_whitespace().rev().collect::<Vec<_>>().join(" ")
}

// Q15 — character frequency
fn char_freq(s: &str) -> HashMap<char, usize> {
    let mut freq = HashMap::new();
    for ch in s.chars() {
        *freq.entry(ch).or_insert(0) += 1;
    }
    freq
}

// Q16 / Q23 — remove duplicates, keep order. Generic: works for chars, ints, strings.
fn remove_duplicates<T: Hash + Eq + Clone>(items: &[T]) -> Vec<T> {
    let mut seen = HashSet::new();
    items.iter().filter(|x| seen.insert((*x).clone())).cloned().collect()
    // HashSet::insert returns false if the value was already there
}

// Q17 — anagram
fn is_anagram(a: &str, b: &str) -> bool {
    let key = |s: &str| {
        let mut chars: Vec<char> = s.to_lowercase().chars().collect();
        chars.sort_unstable();
        chars
    };
    key(a) == key(b)
}

// Q22 — second largest distinct value, one pass. None < Some(x) for every x.
fn second_largest(nums: &[i32]) -> Option<i32> {
    let (mut first, mut second) = (None, None);
    for &n in nums {
        let n = Some(n);
        if n > first {
            second = first;
            first = n;
        } else if n != first && n > second {
            second = n;
        }
    }
    second
}

// Q34 — group by
fn group_by_dept<'a>(employees: &[(&'a str, &'a str)]) -> HashMap<&'a str, Vec<&'a str>> {
    let mut groups: HashMap<&str, Vec<&str>> = HashMap::new();
    for &(name, dept) in employees {
        groups.entry(dept).or_default().push(name);
    }
    groups
}

// Q43 — FizzBuzz with a tuple match
fn fizzbuzz(n: u32) -> String {
    match (n % 3, n % 5) {
        (0, 0) => "FizzBuzz".to_string(),
        (0, _) => "Fizz".to_string(),
        (_, 0) => "Buzz".to_string(),
        _ => n.to_string(),
    }
}

// Q45 — missing number in 1..=n
fn missing_number(nums: &[u64], n: u64) -> u64 {
    n * (n + 1) / 2 - nums.iter().sum::<u64>()
}

// Q50 — pairs that sum to a target, O(n) with a set
fn find_pairs(nums: &[i32], target: i32) -> Vec<(i32, i32)> {
    let mut seen = HashSet::new();
    let mut pairs = Vec::new();
    for &n in nums {
        if seen.contains(&(target - n)) {
            pairs.push((target - n, n));
        }
        seen.insert(n);
    }
    pairs
}

// Q52 — transpose
fn transpose(m: &[Vec<i32>]) -> Vec<Vec<i32>> {
    if m.is_empty() {
        return Vec::new();
    }
    (0..m[0].len()).map(|c| m.iter().map(|row| row[c]).collect()).collect()
}

// Q54 — sort records by salary descending, then name ascending
#[derive(Debug, Clone, PartialEq)]
struct Employee {
    name: &'static str,
    salary: u32,
}

fn main() {
    assert!(is_palindrome_num(1221) && !is_palindrome_num(123) && !is_palindrome_num(-121));
    assert!(is_armstrong(153) && is_armstrong(371) && !is_armstrong(100));
    assert_eq!((1..50).filter(|&n| is_prime(n)).collect::<Vec<_>>(), [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]);
    assert_eq!(fibonacci(10), [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]);
    assert_eq!(factorial(10), Some(3_628_800));
    assert_eq!(factorial(25), None);                      // 25! does not fit in u64
    assert_eq!(digit_sum(-567), 18);
    assert!(is_palindrome_str("A man, a plan, a canal: Panama") && !is_palindrome_str("hello"));
    assert_eq!(count_vowels_consonants("Hello World"), (3, 7));
    assert_eq!(reverse_words("Hello World Python"), "Python World Hello");
    assert_eq!(char_freq("programming")[&'g'], 2);

    let letters: Vec<char> = "programming".chars().collect();
    assert_eq!(remove_duplicates(&letters).iter().collect::<String>(), "progamin");
    assert_eq!(remove_duplicates(&[1, 2, 2, 3, 4, 4, 5, 1]), [1, 2, 3, 4, 5]);

    assert!(is_anagram("Triangle", "Integral") && !is_anagram("hello", "world"));
    assert_eq!(second_largest(&[3, 7, 1, 9, 4, 6, 9, 2]), Some(7));
    assert_eq!(second_largest(&[5, 5, 5]), None);

    let groups = group_by_dept(&[("Asha", "Eng"), ("Ravi", "HR"), ("Meena", "Eng")]);
    assert_eq!(groups["Eng"], ["Asha", "Meena"]);

    assert_eq!((1..=15).map(fizzbuzz).last().unwrap(), "FizzBuzz");
    assert_eq!(missing_number(&[1, 2, 4, 5, 6], 6), 3);
    assert_eq!(find_pairs(&[1, 5, 3, 7, 2, 8, 4], 9), [(7, 2), (1, 8), (5, 4)]);
    assert_eq!(transpose(&[vec![1, 2, 3], vec![4, 5, 6]]), [[1, 4], [2, 5], [3, 6]]);

    let mut staff = vec![
        Employee { name: "Charlie", salary: 80_000 },
        Employee { name: "Alice", salary: 95_000 },
        Employee { name: "Bob", salary: 80_000 },
    ];
    staff.sort_by(|a, b| b.salary.cmp(&a.salary).then_with(|| a.name.cmp(b.name)));
    assert_eq!(staff.iter().map(|e| e.name).collect::<Vec<_>>(), ["Alice", "Bob", "Charlie"]);

    println!("all checks passed");
}
```

**What changed in your thinking, problem by problem**

| Python habit | Rust version | Lesson |
|---|---|---|
| `factorial(25)` just works | Returns `None` | Fixed-size integers: decide what overflow means |
| `second_largest` returns `None` or a number | `Option<i32>` in the signature | The caller is forced to handle "no answer" |
| `float('-inf')` as "nothing yet" | `None`, and `None < Some(x)` | No magic sentinel values |
| `Counter(s)` | `HashMap` + `entry().or_insert(0)` | One hash lookup instead of two |
| `s[::-1]` | `.chars().rev()` | Strings are UTF-8 bytes, characters need an iterator |
| `sorted(emps, key=lambda e: (-e.salary, e.name))` | `sort_by` + `cmp().then_with()` | Comparators compose, no negation trick |
| `x not in seen` then `seen.add(x)` | `seen.insert(x)` returns a bool | One operation, one lookup |

**Practice method that works:** take one problem from your Python file each day, write it in Rust *without* looking here, make the compiler happy, then compare. Twenty problems this way and the borrow checker stops feeling like an enemy.

---

[← 33. FastAPI → axum, plus HTTP client and database](../part-f-web-in-rust/33-fastapi-axum-plus-http-client-and-database.md) · [Index](../README.md) · [35. Compiler errors decoded →](../part-g-practice-and-standing-out/35-compiler-errors-decoded.md)
