---
layout: post
title: Writing a brainfuck interpreter with serialised memory
description: brainfuck is a Turing-complete esolang originally designed to have the smallest possible compiler. This is one of my first stepping stones as I look to writing a compiler for my own language.
---

Invented in 1993 by Urban Müller, brainfuck is a language design to have a very very small compiler. In fact, his compiler was 240 bytes.                        

As I am learning Rust, I thought that writing a compiler for brainfuck would be a great starting challenge. Even without trying my hand at code gold, my final compiler fit in a single file quite neatly and was just under 200 lines (including comments and spacing). The main resource I used for this project is the ![Esolang wiki page](https://esolangs.org/wiki/Brainfuck).

# Environment and syntax

Brainfuck involves manipulating the values of one big array. Traditionally, this array was 30,000 cells long. However, modern day implementations seem to vary in array length and cell capacities (we'll address this later).

To manipulate these array values, brainfuck instructions use a pointer initialised to the first element of the array. The language specifies instructions to move the pointer left and right of the array, as well as incrementing and decrementing the cell value. 

In fact, there are only 8 valid brainfuck instructions, all one character long. All other characters in the source code are ignored as comments. The following is a table describing them.

|   |   |
|---|---|
|Instruction|Description|
|>|Move the pointer right|
|<|Move the pointer left|
|+|Increment the cell|
|-|Decrement the cell|
|.|Output the numeric value of the cell as a character|
|,|Read input and set the cell to the numeric value of the character|
|[|Jump past the matching bracket if the cell value (of the pointer) is 0|
|]|Jump back to the matching bracket if the cell value (of the pointer) is non-zero|

The language supports reading the input of a single character, which numeric value is written to the cell the pointer is on. brainfuck also supports the output of single characters by the cell numeric value, and can also read single character input and set the numeric value of the cell correspondingly.

Notably, the only control flow present in this language are square brackets, which change the execution pointer depending on cell value conditions.

With this set of instructions and an infinite length tape, brainfuck is a Turing-complete language. I think this is quite neat. Armed with this information, I designed a simple interpreter in Rust.

# Implementation
I first setup the interpreter to take in a single argument - the brainfuck source code file name. 

## IO and argument handling in Rust
This was before the days I found out about `clap`, the excellent rust crate designed to manage arguments for CLI tools. I will be using it for future projects (it even generates help menus literate programming style), and it has helped me avoid silly-looking code from this project like this:

``` rust
if args.len() != 2 {
	panic!("[ERROR] Incorrect number of arguments \
			provided. 1 expected, {} provided.",
		   args.len() - 1);
}
```

Once the file was read to memory, I simply filtered out any characters that are not part of the character set (i.e. comments). I found this elegant with Rust's easy predicate/functional programming syntax.

``` rust
const BRAINFUCK_CHARS: [char; 8] = ['>', '<', '+', '-', '.', ',' ,'[',']'];
commands.retain(|c| BRAINFUCK_CHARS.contains(&c));
```

Going with tradition, I set the memory tape size to be 30,000 cells in a mutable vector, and iterated through and executed the instructions. However, the positions of the control flow instructions (i.e. the square brackets) can be precomputed for clarity. 

## Matching brackets and stacks
I used a stack algorithm to compute the matching brackets, returning a vector of `Option<usize>`. For each '\[', its index is push onto the stack, and a '\]' means that the latest index was popped off the stack, and represented the index of the matching bracket. This return a vector of `Option<usize>`, which contained `None` if the instruction in that position is not a square bracket, and `Some(position)` with `position` indicating the matching square bracket index. 

Of course, this necessitates that a vector of the same size as the number of instructions are created, which is inefficient. A tidier solution would be to return this same list without any of the `None` entries. When checking for square brackets, the code simply has to keep track of the current count of brackets seen. This count will yield the index in the matching bracket list.

``` rust
// Compute the positions of the matching bracket for each bracket.  
// Returns a vector of matching bracket positions where relevant.  
fn loop_closures(commands: &Vec<char>) -> Vec<Option<usize>> {  
    let mut stack : Vec<usize> = Vec::new();  
    let mut closure: Vec<Option<usize>> = vec![None; commands.len()];  
    let mut current_index: usize = 0;  
    for command in commands {  
        if command == &'[' {  
            stack.push(current_index.clone())  
        }  
        if command == &']' {  
            assert_ne!(  
                stack.len(),  
                0,  
                "[SYNTAX ERROR] Unmatched loop pairs at position {}.",  
                current_index  
            );  
            let matching_index: usize = stack.pop().unwrap();  
            closure[current_index] = Option::from(matching_index);  
            closure[matching_index] = Option::from(current_index);  
        }  
        current_index += 1;  
    }  
    assert_eq!(  
        stack.len(),  
        0,  
        "[SYNTAX ERROR] Unmatched loop pair(s) at {:#?}",  
        stack  
    );  
    return closure  
}
```

## Interpretation and ASCII
Interpretation was simply done by pattern matching for the right characters and executing the commands. This either involved changing the pointer, the array, or to do IO. 

For the sake of IO, I chose for the array to contain `u8` characters, or $2^8-1=255$ values. This can be easily cast to and from ASCII characters, and I was able to easily implement input:

``` rust
// Read input and set as cell value.
let input_first_char: char = input[0];  
memory[pointer] = input_first_char as u8;
```

and output.

``` rust
// Print current cell value as ASCII char.
 print!("{}", memory[pointer] as char);
```
# Adding serialisation
Soo, that was basically it. All the instructions and the environment needed to interpret brainfuck source code is there. 

However, I wanted to add a little extension to this project, 
and was also not entirely satisfied with the measly 30,000 cells in the memory array. I tried scaling up to the maximum `usize` value, and (expectedly) ran into issues allocating such a large contiguous chunk in memory. Because I wanted to keep things relatively lightweight in memory consumption and I wanted to maximise memory size, I chose to serialise the memory in blocks using the `serde` crate.

To implement this, I checked when the pointer is moved beyond the bounds of the current memory block. This would be the pointer going past `usize::MAX` or going under $0$. If this happens, the current memory block is serialised and stored with its identifier. The memory blocks are tracked using a `u128`, and the memory correct memory block is deserialised into memory or created where needed.

This creates a continuous memory tape which length is `u128::MAX` * 30,000 (very large), successfully increasing the effective length.
# Reflection and extension
This implementation is by no means perfect, and is likely a naive approach to writing an interpreter. However, this was still an extremely useful project where I learnt many text parsing skills and serialisation in Rust.

One performance bottleneck would be if the pointer is consistently moving between the boundary of two memory blocks. This would mean a lot of IO activity and serialisation, which is expensive. To overcome this, keeping three memory blocks in memory would remove this potential performance issue.
