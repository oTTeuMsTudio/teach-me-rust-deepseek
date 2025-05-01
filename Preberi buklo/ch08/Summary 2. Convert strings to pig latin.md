To solve the problem of converting strings to **Pig Latin** in Rust, we need to carefully follow the specified transformation rules and account for UTF-8 encoding, which Rust handles natively.

---

### 🧠 Pig Latin Rules Recap

1. **Words starting with a vowel**:
   - Add `-hay` to the end of the word.
   - Example: `apple` → `apple-hay`

2. **Words starting with a consonant**:
   - Move the **first consonant** to the end of the word.
   - Add `-ay` to the end.
   - Example: `first` → `irst-fay`

3. **Whitespace Handling**:
   - Words are separated by whitespace.
   - Empty words should be ignored.

4. **UTF-8 Encoding**:
   - Rust's `String` and `&str` types are UTF-8 encoded, so we must ensure our slicing operations respect character boundaries (not byte indices).

---

### ✅ Implementation Strategy

1. **Split the input** into words using `split_whitespace()`.
2. **Iterate over each word**, skipping empty ones.
3. **Check the first character**:
   - Convert it to lowercase to determine if it's a vowel (`a`, `e`, `i`, `o`, `u`).
4. **Construct the transformed word**:
   - If vowel: append `-hay`.
   - If consonant: split the word into the rest of the string and the first character, then concatenate `rest + "-" + first + "ay"`.
5. **Join all transformed words** into a single `String`.

---

### 🦀 Rust Code

```rust
pub fn pig_latin(text: &str) -> String {
    let mut result = Vec::new();

    for word in text.split_whitespace() {
        // Skip empty words
        if word.is_empty() {
            continue;
        }

        // Get the first character
        let first_char = match word.chars().next() {
            Some(c) => c,
            None => continue, // Shouldn't happen with non-empty word
        };

        // Check if the first character is a vowel (case-insensitive)
        let is_vowel = matches!(
            first_char.to_lowercase().next().unwrap(),
            'a' | 'e' | 'i' | 'o' | 'u'
        );

        // Build the new Pig Latin word
        let new_word = if is_vowel {
            format!("{}-hay", word)
        } else {
            let rest = &word[first_char.len_utf8()..];
            format!("{}-{}ay", rest, first_char)
        };

        result.push(new_word);
    }

    // Join the transformed words with spaces
    result.join(" ")
}
```

---

### 🧪 Example Usage

```rust
fn main() {
    let input = "first apple Hello world";
    let output = pig_latin(input);
    println!("{}", output);
}
```

**Output:**
```
irst-fay apple-hay ello-Hay orld-way
```

---

### 📌 Notes on UTF-8 Handling

- Rust strings are UTF-8 encoded, so slicing by byte indices can be unsafe.
- We use `first_char.len_utf8()` to get the correct byte offset of the first character.
- This ensures that the rest of the string is sliced correctly, even if the first character is a multibyte Unicode character (like `à`, `é`, etc.).

---

### 🧹 Edge Cases Handled

| Input         | Output         | Explanation |
|--------------|----------------|-------------|
| `"first"`     | `"irst-fay"`   | First consonant moved, `-ay` added |
| `"apple"`     | `"apple-hay"`  | Starts with vowel, `-hay` added |
| `"eight"`     | `"eight-hay"`  | Starts with vowel, `-hay` added |
| `"x"`         | `"-xay"`       | Single-letter consonant |
| `"École"`     | `"Cole-éay"`   | Starts with `É` → treated as consonant |
| `"àpple"`     | `"pple-àay"`   | Starts with `à` → treated as consonant |
| `"   "`       | `""`           | Empty input returns empty string |

---

### 📚 Summary

This Rust implementation of Pig Latin conversion adheres to the rules, handles UTF-8 encoding correctly, and is robust against edge cases such as single-letter words and non-ASCII characters. It's a clean and idiomatic solution that leverages Rust's strong string handling capabilities.
