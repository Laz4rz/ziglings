*I tried to install neovim while doing these, but I failed (.local permissions problems) -- vim ftw.*

This is my approach for the Ziglings from www.ziglings.org -- a series of tiny Zig programs to fix and learn. As always I make dummy-accesible comments and notes as I go through it. Enjoy.


## Walkthrough

1. Hello
   
The structure of a function in Zig is as follows:
```zig
fn name() return_type {
	do_something();
}
```

The functions are private by default, and can be made public by putting `pub` before `fn`. The main has to be explicitly made public.

2. Std
  
Importing is done using:
```zig
const foo = @import("foo");
```

The convention is to store the imports in the constant of the same name as the imported library. More -- imports have to be declared as constants, because they are evaluated at compile time.

Zig allows both `const` and `var` variables. Some things have to be immutable, ie. structs. However if you define some struct as `var`, it may still compile correctly if you never use this struct. This is because the Zig compiler is lazy and will not check on stuff that is defined, but not used.

Above is why you can define imports as `var`, and if you never use them the compiler will not throw an error.

More about evaluating and compiler checking: https://stackoverflow.com/questions/62554187/struct-definition-with-var-instead-of-const-in-zig-language/62567550#62567550

3. Assignment

Numerical variables are created with:
```zig
const x: u8 = 50;
var y: i8 = -50;
```
where u -- unsigned, i -- signed, 8 -- number of bits to store the number. 

In this exercise we are asked to fix the mutability of the variable, number of bits and the signed type. 

The print function in Zig takes two parameters. The first is the string, that may contain placeholders for variables {}, and the second is the 'anonymous list literal', which takes variables that will be put into the placeholders.
```zig
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

7. Strings

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

10. If

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

It can also increment without the :(continue expression).

Third While exercise: introduces `continue` to skip loop iterations. Fourth does `break`.

15. For

Finally `for` is introduced. Can loop over the array elements:

```zig
for (array) |element| {
	do_something()
}
```

We can also do enumeration with `for` loops to iterate with the index. Vanilla enumeration (range) index will have `usize` type, cause its a default type for indexes as we previously established. If we want to operate with this index on some other number we have to cast it. To cast from usize to int we use `@intCast(i)` -- more later.

```zig
    for (bits, 0..) |bit, i| {
        // Note that we convert the usize i to a u32 with
        // @intCast(), a builtin function just like @import().
        // We'll learn about these properly in a later exercise.
        const i_u32: u32 = @intCast(i);
        const place_value = std.math.pow(u32, 2, i_u32);
        value += place_value * bit;
```

@ is used to denote builtin functions.

17. Quiz2
 
Not much. Just previous stuff implemented.

18. Functions

Not main functions incoming -- just define them below main and don't use `pub`. Being private means that the functions is not accesible outside the module (file) in which it's defined.

For a function to take a parameter, define them as:

```zig
fn twoToThe(my_number: u32) u32 {
    return std.math.pow(u32, 2, my_number);
    // std.math.pow(type, a, b) takes a numeric type and two
    // numbers of that type (or that can coerce to that type) and
    // returns "a to the power of b" as that same numeric type.
}
```

20. Quiz3

Nothing hard again. Just remember that when a function does not return anything, you set the return type as void. 

21. (-25) Errors 

We finally get to the error handling. 
- error is a value
- errors are named
- errors come in "error sets", which are a collection of named errors

Below is the most cumbersome error handling implementation in Zig.

Create an error object, with possible error names. It has to be of type constant.
```zig
const NumberError = {
	TooBig,
	TooSmall,
	TooLikeThisNumber,
	TooBad,
};
```

Then we can just return this error as a value from a function or whatever.
```zig
fn doILikeThisNumber(n: u8) NumberError {
	if (n == 3) return NumberError.TooLikeThisNumber;
	return NumberError.TooBad
}
```

And then handle this error, probably later with a `switch` or whatever Zig offers. For now, simply with if comparison.
```zig
if doILikeThisNumber(10) == NumberError.TooBad {
	std.debug.print("Thats not the number I like", .{})
}
```

If you want to say that some variable (or function return) will be either something or something else (probably errorType or correctValueType), then you can use `something!somethingElse` notation, which acts as an Union of types. 

```zig
var my_number: NumberError!u8 = 5;
```

To catch the possible errors, and replace them with default action/value use `catch`. This function will put 6 in the variable if function returns error

```zig
const a: u32 = functionThatCanReturnError(10) catch 6.
```

Ok, we got to 24. errors4 and I got to say I don't like this one. It's like what is this doing and why would it be doing it that way? Whatever. It introduces the similar to `for` over array items notation with catch.

```zig
canFail() catch |err| {
         if (err == FishError.TunaMalfunction) {
            	do_something() 
         }
	do_something_else()
};
```

This pattern is so common that it can be shortened with `try`, which will return the correct output or any error the function returns. This allows going from this:

```zig
fn addFive(n: u32) MyNumberError!u32 {
    const x = detect(n) catch |err| return err;

    return x + 5;
}
```

To this:

```zig
fn addFive(n: u32) MyNumberError!u32 {
    const x = try detect(n);	
    return x + 5;
}
```

Now, this was all needed to use the normal way of printing in Zig. Not, the debug print, but the `stdout.print` that can... fail.

26. Hello2 (the one that can fail, and needs handling)

For the first time we will allow the `main` to return an error. The error type will be automatically infered, which is why we change `void` to `!void`. This is appropriate for main, but may either make some other function hard to write or straight up won't be possible (recursion). More information is at: https://ziglang.org/documentation/master/#Inferred-Error-Sets. 

To use the standard library `stdout` we first need to get it. Only then we can use it to write something to the terminal.  

```zig
const stdout = std.io.getStdOut().writer();
stdout.print("Hello world!\n", .{});
```

Why is this dot here????

27. (-29) defer

Funny one, `defer` prepended to a line of code allows to execute it at the END of the block of code. So i.e. you prepend it to the first line and it makes it execute after all the other ones. 

```zig
pub fn main() void {
    // Without changing anything else, please add a 'defer' statement
    // to this code so that our program prints "One Two\n":
    defer std.debug.print("Two\n", .{});
    std.debug.print("One ", .{});
}
```

Looks to be a really clean way to give certain functions way better readability. If there is a lot of "middle" processing, you can clearly show in the beginning what the function is meant to take in, and give out, and leave the labirinth of middle steps for more careful reading. 

Also an error handling specific `defer` is introduced -- `errdefer`. It does the same thing as `defer`, but only in the case in which the block of code (function) returns an error. Could be used for some kind of cleaning after the error is caught. 

```zig
fn getNumber(i: usize) MyErr!u32 {
	errdefer std.debug.print("Oh no I failed you...", .{});
	var num = try getGetNumber(i);
	num = try getGetGetNumber(i);
	// and milion other ways to fail are all caught with our one cool errdefer
}
```

30. switch

Zig has switches, so cool. Wonder if it has the same construction switches as Scala, or poor-guys-python-switches. Switch basically let's you shorten the big-ass if constructions:

```zig
if (x) {
	doSomething(); // finally switched to correct case for functions
}
else if (y) {
	do2();
}
else {
	do3();
}
```

Becomes:

```zig
switch (c) {
	x => doSomethings();
	y => do2;
	else => do3;
}
``` 

A thing of beauty. It can also return stuff:

```zig
switch (c) {
	x => "itsX";
	else => "notX";
}
```

31. unreachable

`unreachable` keyword is used to signal that some code branch should never be reacehed, and if it is -- it has to be an error.

32. iferror

Error handling of a function value with `if` and `switch` combine. The `value` and `err` are not keywords, but rather arbitrary variable names. This works cause Zig just has this syntax for error unwraping. I dont know, I dont feel like I like it.

```zig
for (nums) |num| {
   std.debug.print("{}", .{num});

    const n = numberMaybeFail(num);
    if (n) |value| {
        std.debug.print("={}. ", .{value});
    } else |err| switch (err) {
        MyNumberError.TooBig => std.debug.print(">4. ", .{}),
        MyNumberError.TooSmall => std.debug.print("<4. ", .{})
    }
}
```

33. Quiz 4

Needed this error handling syntax:

```zig
    if (my_num) |value| {
        try stdout.print("my_num={}\n", .{value});
    }
    else |err| switch (err) {
        NumError.IllegalNumber => std.debug.print("Dupa", .{}),
    }
```

34. (-36) enums

Zig also has enums. You can define a type that can only take predefined values, therefor you do not need to assign arbitrary numbers to represent some operation, you can just name it using an `enum`.

```zig
const Ops: = enum {inc, dec, pow};

// rest of program

switch (op) {
Ops.inc => something(),
// rest
}
```

Enums by construction are associated with a number. We can either rely on the automatic assignment (that can be checked with @intFromEnum(MyEnum.foo)), or assign it by hand.

```zig
const Stuff = enum(u8){ foo = 16, boo = 0x00ff00 } // 0x00ff00 is a hex format, where each two digits represent a bit between 0-255;
```

Also string formatting that is put inside the format placeholders:

```zig
    //     {x:0>6}
    //      ^
    //      x       type ('x' is lower-case hexadecimal)
    //       :      separator (needed for format syntax)
    //        0     padding character (default is ' ')
    //         >    alignment ('>' aligns right)
    //          6   width (use padding to force width)
```

37. (-38) structs

Naturally, after predefined structs (i think they are), like `error` and `enum`, we can also create custom structs. 

```zig
const Character = struct {
	role: Role, // defined earlier enum
	health: u8,
}

var the_cool_guy = Character{
	.role = Role.Chad,
	.health = 100,
}

the_cool_guy.health -= 5;
```

We can also add the characters to an array of structs.

```zig
var chars: [2]Character = undefined; // standard way to define empty variables

chars[0] = Character{
    .role = Role.wizard,
    .gold = 20,
    .health = 100,
    .experience = 10,
};
```
Now the above array has an undefined value on index 1. If we try to access this struct or its attributes we are going to get some random "garbage" values.

39. (-43) pointers

```zig
var foo: u8 = 5;      // foo is 5
var bar: *u8 = &foo;  // bar is a pointer
const too: u8 = 5;
const bot: *const u8 = &too; 
// You can always make a const pointer to a mutable value (var), but
// you cannot make a var pointer to an immutable value (const).
// This sounds like a logic puzzle, but it just means that once data
// is declared immutable, you can't coerce it to a mutable type.
// Think of mutable data as being volatile or even dangerous. Zig
// always lets you be "more safe" and never "less safe."
//     u8         the type of a u8 value
//     foo        the value 5
//     *u8        the type of a pointer to a u8 value
//     &foo       a reference to foo
//     bar        a pointer to the value at foo
//     bar.*      the value 5 (the dereferenced value "at" bar)

var boo: u8 = undefined;
boo = foo.*;
std.debug.print("{}", .{boo});
```

Ok now I know whats the deal with constant pointer types.

```zig
var   foo: u8 = 5;
const boo: u8 = 2;

var vpfoo: *u8 = &foo;          // pointer to the foo address, can be changed to point to another variable, can be used to change the value of foo
const cpfoo: *u8 = &foo;        // pointer to the foo address, can not be changed to point to another variable, can be used to change the value of foo
var cvpfoo: *const u8 = &foo;   // pointer to the foo address, can be changed to point to another variable, can not be used to change the value of foo
const ccpfoo: *const u8 = &foo; // pointer to the foo address, can not be changed to point to another variable, can not be used to change the value of foo

var vpboo: *u8 = &boo;          // can not be created, would result in a mutable value when dereferencing which would contradict the const type of boo
const cpboo: *u8 = &boo;	// also can not be created, would result in an immutable pointer, but mutable dereference, also contradicts
var cvpboo: *const u8 = &boo;   // pointer to the boo address, can be changed to point to another variable, can not be used to change the value of boo
const ccpboo: *const u8 = &boo; // pointer to the boo address, can not be changed to point to another variable, can not be used to change the value of boo
```

Writing this took my 10 minutes, and I think it's the most I learned about pointers since highschool.

```zig
var x: u8 = 5;
var px: *u8 = &x;

// px   -- address of x
// &px  -- address of px 
// px.* -- accessed variable x (dereferenced), can be used to write if not *const pointer type 
```

We can also combine pointers with structs. Important remark is that counterintuitively we do not need to dereference the struct pointer to access it's elements. 

```zig
// YES: my_struct_pointer.x 
// NO:  my_struct_pointer.*.x

const Class = enum {
	wizard,
	knight,
	bard,
}

const Character = struct {
	class: Class,
	gold: u32,
	health: u8 = 100,
	experience: u32,
	// we can also mention a mentor of this character, but he can also be non-existent
	// so we can type a pointer to a struct as *Struct
	// and to allow it to be null, we have to add ? before it, why not undefined instead of null???
	mentor ?*Character = null,
}

var crodor = ...

var glorp = Character {
	.class      = Class.wizard,
	.gold       = 100,
	.experience = 20,
	.mentor     = &krodor // some other character idc  
}

// then we can pass a pointer to this character to a function as c and use it, for example:
// notice how we do not need to specify enum name for each of enum values
const class_name = switch (c.class) {
    .wizard => "Wizard",
    .thief => "Thief",
    .bard => "Bard",
    .warrior => "Warrior",
};
```

We can also check if value is not `null`, with the funny if error notation seen previously.

```zig
if (c.mentor) |mentor| {
	std.debug.print("Mentor: ", .{});
	printCharacter(mentor); // some function to parse and print the Character struct
}
```

44. Quiz5

Quicky  introduction of linked lists, by elephants holding tails -- cute. 

45. 
