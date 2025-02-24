To print the lyrics of "The Twelve Days of Christmas" in Rust while taking advantage of the repetition, we can use a structured approach. The song has a repetitive structure where each verse builds upon the previous one. We'll store the gifts in an array or vector and iteratively construct the lyrics for each verse.

Here’s how you can implement this in Rust:

### Step-by-Step Explanation:
1. **Store the gifts**: Create a list (vector) of the gifts corresponding to each day.
2. **Iterate through the days**: Use a loop to iterate from the first day to the twelfth day.
3. **Build the verse**: For each day, construct the verse by appending the current day's gift and all the previous gifts.
4. **Print the verse**: Print the constructed verse for each day.

Below is the complete implementation:

```rust
fn main() {
    // List of gifts for each day
    let gifts = vec![
        "a Partridge in a Pear Tree",
        "two Turtle Doves",
        "three French Hens",
        "four Calling Birds",
        "five Gold Rings",
        "six Geese-a-Laying",
        "seven Swans-a-Swimming",
        "eight Maids-a-Milking",
        "nine Ladies Dancing",
        "ten Lords-a-Leaping",
        "eleven Pipers Piping",
        "twelve Drummers Drumming",
    ];

    // Ordinal numbers for the days
    let ordinals = [
        "first", "second", "third", "fourth", "fifth", "sixth", "seventh", "eighth",
        "ninth", "tenth", "eleventh", "twelfth",
    ];

    // Loop through each day
    for i in 0..12 {
        println!("On the {} day of Christmas, my true love gave to me:", ordinals[i]);

        // Build the list of gifts for the current day
        for j in (0..=i).rev() {
            if j == 0 && i != 0 {
                // Special case for "and" before the Partridge
                print!("and ");
            }
            println!("{}", gifts[j]);
        }

        println!(); // Add a blank line between verses for readability
    }
}
```

### Key Points in the Code:
1. **Gifts Vector**: The `gifts` vector contains all the gifts in order. This allows us to access the appropriate gift for each day using its index.
2. **Ordinal Numbers Array**: The `ordinals` array holds the ordinal names for the days ("first", "second", etc.), which are used in the lyrics.
3. **Reversed Iteration**: When printing the gifts for each day, we iterate in reverse order (`(0..=i).rev()`), starting from the current day back to the first day.
4. **Special Case for "and"**: On days after the first, the word "and" is added before "a Partridge in a Pear Tree".

### Output:
When you run the program, it will print the full lyrics of "The Twelve Days of Christmas," with proper formatting and repetition. Here’s an excerpt of the output:

```
On the first day of Christmas, my true love gave to me:
a Partridge in a Pear Tree

On the second day of Christmas, my true love gave to me:
two Turtle Doves
and a Partridge in a Pear Tree

On the third day of Christmas, my true love gave to me:
three French Hens
two Turtle Doves
and a Partridge in a Pear Tree

...

On the twelfth day of Christmas, my true love gave to me:
twelve Drummers Drumming
eleven Pipers Piping
ten Lords-a-Leaping
nine Ladies Dancing
eight Maids-a-Milking
seven Swans-a-Swimming
six Geese-a-Laying
five Gold Rings
four Calling Birds
three French Hens
two Turtle Doves
and a Partridge in a Pear Tree
```

This implementation efficiently leverages Rust's features to handle the repetitive structure of the song while maintaining clarity and readability.
