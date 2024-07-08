*I tried to install neovim while doing these, but I failed (.local permissions problems) -- vim ftw.*

This is my approach for the Ziglings from www.ziglings.org -- a series of tiny Zig programs to fix and learn. As always I make dummy-accesible comments and notes as I go through it. Enjoy.


## Walkthrough

1. Hello
The structure of a function in Zig is as follows:
```
fn name() return_type {
	do_something();
}
```

The functions are private by default, and can be made public by putting `pub` before `fn`. The main has to be explicitly made public.

2. Std
Importing is done using:
```
const foo = @import("foo");
```

The convention is to store the imports in the constant of the same name as the imported library. More -- imports have to be declared as constants, because they are evaluated at compile time.

Zig allows both `const` and `var` variables. Some things have to be immutable, ie. structs. However if you define some struct as `var`, it may still compile correctly if you never use this struct. This is because the Zig compiler is lazy and will not check on stuff that is defined, but not used.

Above is why you can define imports as `var`, and if you never use them the compiler will not throw an error.

More about evaluating and compiler checking: https://stackoverflow.com/questions/62554187/struct-definition-with-var-instead-of-const-in-zig-language/62567550#62567550

3. Assignment
Numerical variables are created with:
```
const x: u8 = 50;
var y: i8 = -50;
```
where u -- unsigned, i -- signed, 8 -- number of bits to store the number. 

In this exercise we are asked to fix the mutability of the variable, number of bits and the signed type. 

The print function in Zig takes two parameters. The first is the string, that may contain placeholders for variables {}, and the second is the 'anonymous list literal', which takes variables that will be put into the placeholders.
```
const x: u8 = 1
print("Hello {}", .{x};
```

4. Arrays
The Zig arrays have a constant length. The also need to come with type on the right side. Which I do not understand yet. 
```zig 
var foo: [3]u32 = [3]u32{1, 2, 3};
```
You see how strange? Type on both sides. 

Strangely in the same example, it can also be infered, same as length:
```zig
var foo = [_]u32{ 42, 108, 5423 };
```

Also values can be both accessed and assigned with existing indexes, and their length through `len` method.
```zig
foo[2] = 16;;
const bar = foo[2];
const length = foo.len;
```

5. Arrays2
Zig has "fun" (sic!) array operators. Use `++` to concatenate arays, and `**` to repeat arrays. This only works in comptime, which the time when the program is _being compiled_. 

6. Strings
Zig strings are just bytes arrays.
```zig
const foo = "Hey";
const boo = [_]u8{'H', 'e', 'y'};
```
With the "" -- strings, '' -- characters distinction. 

To correctly print the characters and strings instead of their decimal representation use `u` and `s` inside the print placeholders respectively. This will tell Zig to print them as UTF-8 characters. Using `c` (ASCII characters) will work for the first 128 UTF-8 characters, as they are the same between encodings.

7. Strings2
Zig has multiline strings. With this strange comment like notation:
```zig
const two_lines = 
	\\ one line
	\\ two line
;
```

8. Quiz
The idiomatic type for array indexing variables is `usize`. The exact size for this type is CPU architecture dependent.

9. If 
Zig uses classic comparison operators. The difference with Zig;s if statement is that it wont accept types other than bool. 

```zig
if some_bool {
	do_something_1
} else
{
	do_something_2
}
```

10. If2
if statements are also viable in the variable definitions. 

```zig
const foo: u8 = if (a) 2 else 3;
```

11. While 
Vim is starting the infuriate me a little. Zig for some reason has a continue expression statement thats optional for a while loop. Why? It could've just been inside the while??? Either way, continue runs everytime the loop continues. 
 
```zig
var foo = 2;
while (foo < 10) : (foo += 2) {
	do_something()
}
```


