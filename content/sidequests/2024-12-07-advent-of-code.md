+++
title = "Advent of Code 2024"
+++
It's that time of year again...

***⁺₊❆❆Christmas!❆❆⁺₊***

Which means another advent calendar of coding problems to do! 
This page will detail my solutions to the problems and my thought processes for each day.
By the way I will be writing all my solutions in rust.

---
## Index
 - [Day 1](#day-1-historian-histeria)
 - [Day 2](#day-2-red-nosed-reports)
 - [Day 3](#day-3-mull-it-over)
 - [Day 4](#day-4-ceres-search)
---
## [Day 1 - Historian Histeria](https://adventofcode.com/2024/day/1)
### Part 1
*The problem:*
Two lists of numbers are given, we need to find the sum of all differences between the smallest numbers of each list in order.

*My solution:*
This one's easy, we just have to sort both lists then iterate through both of them, finding the difference between the elements' of both lists in order.

### Part 2
*The problem:*
We have the same two lists, now we just have to find the summation of all instances of a number in the left list multiplied by the amount it occurs in the right list.

*My solution:*
Again, this is easy, we need to use either a HashMap or a table to keep track of the instances of each number in the list on the right. 
After we iterate through the right list and have our HashMap, we can iterate through the list on the left and multiply each number by the value in the HashMap.

### Code
```rust
use std::{collections::HashMap, io::Read};
pub fn solve() -> anyhow::Result<()> {
    let mut buffer: Vec<u8> = Vec::new();
    let mut stdin = std::io::stdin();
    stdin.read_to_end(&mut buffer)?;

    let list = String::from_utf8(buffer)?;
    let mut distance_count = 0;

    let mut first_list: Vec<isize> = Vec::new();
    let mut second_list: Vec<isize> = Vec::new();
    list.split("\n").for_each(|list_entry| {
        println!("{}", list_entry);
        let split_entry: Vec<&str> = list_entry.split_whitespace().collect();

        if split_entry.len() == 2 {
        let first_num = isize::from_str_radix(split_entry[0], 10).unwrap();
        let second_num = isize::from_str_radix(split_entry[1], 10).unwrap();

            first_list.push(first_num);
            second_list.push(second_num);
        }
    });

    first_list.sort();
    second_list.sort();

    for index in 0..first_list.len() {
        distance_count += (first_list[index] - second_list[index]).abs();
    }

    let mut right_num_count: HashMap<isize, isize> = HashMap::new();

    for num in second_list.iter() {
        match right_num_count.get_mut(num) {
            Some(old) => *old = *old + 1,
            None => {
            right_num_count.insert(*num, 1);
            }
        }
    }

    let mut similarity_score = 0;
    for num in first_list {
        if let Some(matching_count) = right_num_count.get(&num) {
            similarity_score += num * *matching_count;
        }
    }

    println!("Full distance: {distance_count}");
    println!("Similarity Score: {similarity_score}");
    Ok(())
}
```
## Day 2 - Red-Nosed Reports
### Part 1
*The Problem:*
We are given a list of lists of numbers. For each sub-list we have to check to see if each list is either increasing or decreasing by a difference of one or two.
Each list that fulfills both requirements is counted, and we submit the count at the end.

*My Solution:*
Another kind of straightforward solution, just go through each line and if the absolute difference between one digit and the next is less than or equal to two.
I also have a funny way of keeping track of the direction, where I just store the first initial difference, if its negative I keep checking to see if the next difference is negative and vice versa.

### Part 2
*The Problem:*
The question is essentially the same, except, we can take any digit out of a sub-list if it doesn't work, 
and if the new sub-list with the element removed works, we can count it.

*My solution*
We keep most of the solution from the last part, the only change is that if we find some issue with a sub-list we check three different iterations of that sub-list.
I wrote a function that checks for sub-list validity and returns the index at which a fail state occurred if it does. So I thought, it must be the case that if we fail at a specific index in a list, 
and if there is only one number that will cause a fail state, then the element that causes the fail state is either at the index or right next to it.
So I just run my validity checker function on the sub-list with either the index - 1, index, and index + 1 removed.

### Code
```rust
use std::io::stdin ;
pub fn solve() -> anyhow::Result<()>{
    fn test_line(nums: &Vec<isize>) -> Option<isize> {
        let mut trend = 0isize;
        for index in 0..(nums.len() - 1) {
            //println!("Index: {index}, is_safe: {is_safe}, trend: {trend}");
            let abs_diff = nums[index].abs_diff(nums[index + 1]);
            let true_diff = nums[index] - nums[index + 1];
            if abs_diff == 1 || abs_diff == 2 || abs_diff == 3 {
                let is_safe = match trend {
                    ..0 => true_diff < 0,
                    0 => { 
                        trend = true_diff;
                        true
                    },
                    1.. =>  true_diff > 0
                };

                if !is_safe {
                    return Some(index as isize);
                }
            }
            else {
               return Some(index as isize);
            }
        }
        return None;
    }
    let line_buf = stdin().lines();
    let mut base_safe_count = 0;
    let mut adjusted_safe_count = 0;
    for line in line_buf {
        let nums : Vec<isize> = line?.split_whitespace().map(|num_str| isize::from_str_radix(num_str, 10).unwrap()).collect();

        if let Some(index) = test_line(&nums) {
            let (left_slice, right_slice)= &nums.split_at(usize::try_from(index)?);
            let mut valid = test_line(&[left_slice, &right_slice[1..]].concat()).is_none();


            if index > 0 && !valid {
                let (left_slice, right_slice)= &nums.split_at(usize::try_from(index - 1)?);
                valid = test_line(&[left_slice, &right_slice[1..]].concat()).is_none();
            }
            if index > 1 && !valid {
                let (left_slice, right_slice)= &nums.split_at(usize::try_from(index - 2)?);
                valid = test_line(&[left_slice, &right_slice[1..]].concat()).is_none();
            }
            if usize::try_from(index)? < nums.len() - 1 && !valid {
                let (left_slice, right_slice)= &nums.split_at(usize::try_from(index + 1)?);
                valid = test_line(&[left_slice, &right_slice[1..]].concat()).is_none();
            }

            if valid {
                adjusted_safe_count+= 1;
            }
        }
        else {
            adjusted_safe_count += 1;
            base_safe_count += 1;
        }

    }

    println!("Safe count: {base_safe_count}");
    println!("Adjusted Safe count: {adjusted_safe_count}");

    Ok(())
}
```
## Day 3 - Mull It Over
### Part 1
*The Problem:*
We are given text and asked to find a specific pattern inside the text. 
For each pattern 'mul(x, y)', we add the product of x and y to a running sum. At the end we submit the running sum.

*My Solution:*
Ok so I think this was at least a little less straightforward than the past two, at least it is in rust without using any other crates (I think).
Maybe there are better pattern matching things I could have used. Now we are asked to parse through a text file and look for a specific pattern. 
We could technically write a lexer to create tokens for what we are looking for, but that would probably be major overkill.
So I just iterate through the text and call a function I named try_parse which returns an error if it could not successfully parse the pattern,
if we do successfully parse the pattern, we return the pair. Here's the functions, we check if the current index matches the pattern piece by piece,
early returning every time we don't find a character matching the pattern.

```rust
fn try_parse(slice: &[char], cursor: &mut usize) -> anyhow::Result<(isize, isize)> {
    if !(slice[*cursor] == 'm' && slice[*cursor + 1] == 'u' && slice[*cursor + 2] == 'l' && slice [*cursor + 3] == '(') {
        bail!("Does not match");
    }
    *cursor += 4; // Advance the cursor at every step to not go over the same character twice.

    let num1 = parse_num(slice, cursor)?;
    if slice[*cursor] != ',' { 
        bail!("Could not find comma");
    }
    *cursor +=1;

    let num2 = parse_num(slice,cursor)?;

    if slice[*cursor] != ')' {
        bail!("Could not find closing paren");
    }
    *cursor += 1;

    Ok((num1, num2))
}

fn parse_num(slice: &[char], cursor: &mut usize) -> anyhow::Result<isize> {
    let mut char : char = slice[*cursor];

    if !char.is_digit(10) {
        anyhow::bail!("Not a number");
    }

    let mut num = 0;

    while char.is_digit(10) {
        num = num * 10 + char.to_digit(10).context("Could not conver char to digit")?;
        *cursor += 1;
        if *cursor >= slice.len() {
            bail!("Index out of bounds");

        }
        char = slice[*cursor];
    }

    Ok(isize::try_from(num)?)
}
```


### Part 2
*The Problem:*
There are two new patterns we have to look for **do()** and **don't()**. 
Each time we see a **do()** we start checking for the old pattern again and each time we see a **don't()** we stop checking for the old pattern.

*My Solution:*
So I did a bit of a code rewrite for this (should I show my iterations for each problem). I think I had to rewrite my try_parse function, 
I don't have the old version though and I don't remember the code either (╥﹏╥). Either way I attempt to parse both **do()** and **don't** before I try to parse **mul(x,y)**.
Parsing **do()** sets enabled to true, parsing **don't** sets enabled to false, if the enabled variable is false we don't even attempt to run try_parse. 
One of the first things I did was choose to convert the text from a rust String to a Vec&lt;char&gt;, it's easier to iterate through it.
Now I can't compare a Vec&lt;char&gt; to a string like "do()", so I had to write a function that let me do that.

```rust
fn match_chars(slice: &[char], value: &str) -> bool {
    let value_chars : Vec<char> = value.chars().collect();
    for index in 0..value.len() {
        if slice[index] != value_chars[index] {
            return false;
        }
    }
    return true;

}
```

### Code 
```rust
use std::io::Read;
use anyhow::{bail, Context};
pub fn solve() -> anyhow::Result<()> {
    let mut buffer: Vec<u8> = Vec::new();
    let mut stdin = std::io::stdin();
    stdin.read_to_end(&mut buffer)?;
    let full_text : Vec<char> = String::from_utf8(buffer)?.chars().collect();
    let mut cursor = 0;
    let mut total = 0;
    let mut enabled = true;
    while cursor < full_text.len() {
        match try_parse(&full_text, &mut cursor) {
            Ok(pair) => total = total + pair.0 * pair.1,
            Err(_) => cursor += 1
        }
    }

    let mut adjusted_total = 0;
    let mut cursor = 0;
    while cursor < full_text.len() {
        if cursor < full_text.len() - 4 && match_chars(&full_text[cursor..], "do()") {
            enabled = true;
            cursor += 4;
        }
        else if cursor < full_text.len() - 7 && match_chars(&full_text[cursor..], "don't()") {
            enabled = false;
            cursor += 7;

        }
        else if enabled {
            match try_parse(&full_text, &mut cursor) {
                Ok(pair) => adjusted_total = adjusted_total + pair.0 * pair.1,
                Err(_) => cursor += 1
            }
        }
        else {
            cursor += 1;
        }
    }

    println!("Total: {total}");
    println!("Adjusted Total: {adjusted_total}");
    Ok(())
}
```

## Day 4 - Ceres Search
### Part 1
*The Problem:*
We are given some text (or a 2d char array) and we have to find and count every instance of the string **XMAS** in any direction.

*The Solution:*
I wrote a function that accepts two direction, a reference to a char slice, and position data. 
It checks to see if the position + the direction vector times three is out of bounds of the 2d array. 
If it's not, we can check to see if each step away from our initial point matches the corresponding character in **XMAS**.

```rust
fn has_xmas(y_dir: isize, x_dir: isize, char_array: &Vec<Vec<char>>, x: usize, y:usize ) -> anyhow::Result<bool> {
    let width = char_array[0].len();
    let height = char_array.len();

    if char_array[y][x] != 'X' {
        return Ok(false);
    }

    let iy = isize::try_from(y)?;
    let ix = isize::try_from(x)?;
    let iwidth = isize::try_from(width)?;
    let iheight = isize::try_from(height)?;

    // Bounds Check
    if iy  + y_dir * 3< 0isize || ix + x_dir * 3< 0isize || ix + x_dir * 3> iwidth - 1|| iy + y_dir * 3> iheight - 1 {
        return Ok(false);
    }

    const MAS: [char;3]= ['M', 'A', 'S'];
    for index_step in 1..4 {
        let y_index = usize::try_from(iy + y_dir * index_step)?;
        let x_index = usize::try_from(ix + x_dir * index_step)?;
        if char_array[y_index][x_index] != MAS[usize::try_from(index_step - 1)?] {
            return Ok(false);
        }
    }
    
    return Ok(true);
}

```
### Part 2
*The Problem:*
It turns our we weren't looking for the string **XMAS**, but instead we were looking for each instance of the string **MAS** in the shape of an X (did Elon Musk write this question?).

*The Solution:*
So I wrote two different functions for this part. The first was a function to compare a char slice to a 3x3 char array. 
Pretty basic stuff, we go from 0 to 3 in both the x and y directions comparing each element in char array at those indices to the element at the char slice at those indices + an offset.
If the element in the char array is a space we ignore it, otherwise its a direct comparison.
```rust
fn compare_matrix(matrix: [[char; 3]; 3], char_array: &Vec<Vec<char>>, x: usize, y: usize) -> bool {
    for y_offset in 0..3 {
        for x_offset in 0..3 {
            if matrix[y_offset][x_offset] != ' ' && 
            matrix[y_offset][x_offset] != char_array[y + y_offset][x + x_offset] {
                return false;
            }
        }
    }
    return true;
}
```
Next I wrote has_x_mas() where we run the compare_matrix() function at one position with four different char_arrays.
I was going to do some fancy matrix rotation stuff but it wasn't really neccessary. It turns out **MAS** in the shape of an x can only be expressed in four ways...

```
M M | S M | S S | M S
 A  |  A  |  A  |  A 
S S | S M | M M | M S
```
So I just hardcoded each matrix and ran the compare matrix function on each.
```rust
fn has_x_mas(char_array: &Vec<Vec<char>>, x: usize, y:usize ) -> anyhow::Result<bool> {
    let width = char_array[0].len();
    let height = char_array.len();


    if x + 2> width - 1|| y + 2> height - 1 {
        return Ok(false);
    }

    const MAS1: [[char; 3];3]= [
        ['M', ' ', 'M'],
        [' ', 'A', ' '],
        ['S', ' ', 'S']
    ];
    const MAS2: [[char; 3];3]= [
        ['M', ' ', 'S'],
        [' ', 'A', ' '],
        ['M', ' ', 'S']
    ];
    const MAS3: [[char; 3];3]= [
        ['S', ' ', 'M'],
        [' ', 'A', ' '],
        ['S', ' ', 'M']
    ];
    const MAS4: [[char; 3];3]= [
        ['S', ' ', 'S'],
        [' ', 'A', ' '],
        ['M', ' ', 'M']
    ];

    return Ok(
        compare_matrix(MAS1, char_array, x, y) ||
        compare_matrix(MAS2, char_array, x, y) ||
        compare_matrix(MAS3, char_array, x, y) ||
        compare_matrix(MAS4, char_array, x, y)
    );
}
```

### Code
```rust
use crate::utils::get_input;
pub fn solve() -> anyhow::Result<()> {
    let input = get_input()?;
    let char_array : Vec<Vec<char>> =  input.split("\n").map(|line| line.chars().collect()).filter(|arr: &Vec<char>| arr.len() != 0).collect();

    let mut xmas_count = 0;
    let mut x_mas_count = 0;
    for x in 0..char_array[0].len() {
        for y in 0..char_array.len() {

            if let Ok(true) = has_xmas(1, 1, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(1, -1, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(-1, -1, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(-1, 1, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(0, 1, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(0, -1, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(1, 0, &char_array, x, y) {
                xmas_count += 1;
            }
            if let Ok(true) = has_xmas(-1, 0, &char_array, x, y) {
                xmas_count += 1;
            }

            if let Ok(true) = has_x_mas(&char_array, x, y) {
                x_mas_count += 1;
            }
        }
    }
    println!("XMAS COUNT: {xmas_count}");
    println!("X-MAS COUNT: {x_mas_count}");

    Ok(())
}
```

## Day 5 - Print Queue
# Part 1
*The Problem:*
Input is given in two parts, the first part consists of lines that describe the order of numbers. The number on the left of a line must come before the number on the right in the second part.
The second part of the input consists of another list of lines that have comma seperated numbers. For every line the correctly abides by the rules described in the first part of the input,
we take the middle number of each line and add it to a running sum.

*The Soltions:*
*Jeopardy Noises...*
I am kind of stumped for a non-brute force solution to this. I imagine there are no circular references in the ordering description.
This means that there is a number that is referenced only once in the ordering description.
