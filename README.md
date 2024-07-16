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

---

2. Std

Importing is done using:
```zig
const foo = @import("foo");
```

The convention is to store the imports in the constant of the same name as the imported library. More -- imports have to be declared as constants, because they are evaluated at compile time.

Zig allows both `const` and `var` variables. Some things have to be immutable, ie. structs. However if you define some struct as `var`, it may still compile correctly if you never use this struct. This is because the Zig compiler is lazy and will not check on stuff that is defined, but not used.

Above is why you can define imports as `var`, and if you never use them the compiler will not throw an error.

More about evaluating and compiler checking: https://stackoverflow.com/questions/62554187/struct-definition-with-var-instead-of-const-in-zig-language/62567550#62567550

---

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

---

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

---

5. Arrays2

Zig has "fun" (sic!) array operators. Use `++` to concatenate arays, and `**` to repeat arrays. This only works in comptime, which the time when the program is _being compiled_.

---

7. Strings

Zig strings are just bytes arrays.
```zig
const foo = "Hey";
const boo = [_]u8{'H', 'e', 'y'};
```
With the "" -- strings, '' -- characters distinction.

To correctly print the characters and strings instead of their decimal representation use `u` and `s` inside the print placeholders respectively. This will tell Zig to print them as UTF-8 characters. Using `c` (ASCII characters) will work for the first 128 UTF-8 characters, as they are the same between encodings.

---

8. Strings2

Zig has multiline strings. With this strange comment like notation:
```zig
const two_lines =
	\\ one line
	\\ two line
;
```

--- 

9. Quiz

The idiomatic type for array indexing variables is `usize`. The exact size for this type is CPU architecture dependent.

---

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

---

11. If2

if statements are also viable in the variable definitions.

```zig
const foo: u8 = if (a) 2 else 3;
```

12. While

Vim is starting the infuriate me a little. Zig for some reason has a continue expression statement thats optional for a while loop. Why? It could've just been inside the while??? Either way, continue runs everytime the loop continues.

```zig
var foo = 2;
while (foo < 10) : (foo += 2) {
	do_something()
}
```

It can also increment without the :(continue expression).

Third While exercise: introduces `continue` to skip loop iterations. Fourth does `break`.

--- 

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

---

17. Quiz2

Not much. Just previous stuff implemented.

---

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

---

20. Quiz3

Nothing hard again. Just remember that when a function does not return anything, you set the return type as void.

---

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

---

26. Hello2 (the one that can fail, and needs handling)

For the first time we will allow the `main` to return an error. The error type will be automatically infered, which is why we change `void` to `!void`. This is appropriate for main, but may either make some other function hard to write or straight up won't be possible (recursion). More information is at: https://ziglang.org/documentation/master/#Inferred-Error-Sets.

To use the standard library `stdout` we first need to get it. Only then we can use it to write something to the terminal.

```zig
const stdout = std.io.getStdOut().writer();
stdout.print("Hello world!\n", .{});
```

Why is this dot here????

---

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

---

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

---

31. unreachable

`unreachable` keyword is used to signal that some code branch should never be reacehed, and if it is -- it has to be an error.

---

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

---

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

---

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

---

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

---

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

---

44. Quiz5

Quicky introduction of linked lists, by elephants holding tails -- cute.

---

45. (-46) optionals

Zig allows creating optionals -- variables that can either hold some value or a `null`.

```zig
var foo: ?u32 = 10;
foo = null;

var boo = foo orelse 2;
// this will either assign value if foo is not null
// or 2 by default if foo is null
// therefore we can be sure that boo is now u32 type

var coo = foo.?;
// short for `orelse unreachable`
// i guess its for when we really need  this value
```

The optionals are similar to union of error!type variable.

```zig
var maybe_bad: Error!u32 = Error.Evil;
var number: u32 = maybe_bad catch 0;
```

---

46. (-47) methods

Zig structs can have methods attached to them (almost classes, almost). You create them by:

```zig
const Foo = struct {
	pub fn hello() void }
		std.debug.print("Hello from Foo\n", .{});
	}
}

Foo.hello();
// method is defined inside the Foo namespace
// which is why it's called with namespace.method
```

If the first argument of the method is of the namespace type (ie. struct) or a pointer to it, then it acts as `self` keyword and allows self-calling on variable of this type.

```zig
const Bar = struct{
	pub fn a(self: Bar) void {}
	pub fn b(this: *Bar, other: u8) void {}
	pub fn c(bar: *const Bar) void {}
 };
// there is no one way of naming it
// three notations above are popular

var bar = Bar{};
bar.a()  // is equivalent to Bar.a(bar)
bar.b(3) // is equivalent to Bar.b(&bar, 3)
bar.c()  // is equivalent to Bar.c(&bar)
```

Method a: Pass by Value

The entire Bar struct is copied when the method is called.
Changes to self inside the method don't affect the original Bar instance.
Use this when you don't need to modify the struct and it's small enough that copying is not a performance concern.


Method b: Mutable Pointer

Passes a pointer to the Bar instance, allowing modifications to the original struct.
More efficient for large structs as it avoids copying.
Use this when you need to modify the struct's fields.


Method c: Const Pointer

Passes a pointer to the Bar instance, but doesn't allow modifications.
Efficient like b, but ensures the method doesn't change the struct.
Use this when you need to read from a large struct but want to ensure it's not modified.

Enums can also have methods.

```zig
pub const FileExt = enum {
    c,
    cpp,
    h,
    ll,
    bc,
    assembly,
    shared_library,
    object,
    static_library,
    zig,
    zir,
    unknown,

    pub fn clangSupportsDepFile(ext: FileExt) bool {
        return switch (ext) {
            .c, .cpp, .h => true,

            .ll,
            .bc,
            .assembly,
            .shared_library,
            .object,
            .static_library,
            .zig,
            .zir,
            .unknown,
            => false,
        };
    }
};
```

---

50. Quiz 6

We are finally writing something ourselves. Just replicate the tail methods and you're good. Also, I managed to finally turn on vim zig and markdown highlighting -- noice.

---

51. no value

```zig
// Zig has at least four ways of expressing "no value":
//
// * undefined
//
//       var foo: u8 = undefined;
//
//       "undefined" should not be thought of as a value, but as a way
//       of telling the compiler that you are not assigning a value
//       _yet_. Any type may be set to undefined, but attempting
//       to read or use that value is _always_ a mistake.
//
// * null
//
//       var foo: ?u8 = null;
//
//       The "null" primitive value _is_ a value that means "no value".
//       This is typically used with optional types as with the ?u8
//       shown above. When foo equals null, that's not a value of type
//       u8. It means there is _no value_ of type u8 in foo at all!
//
// * error
//
//       var foo: MyError!u8 = BadError;
//
//       Errors are _very_ similar to nulls. They _are_ a value, but
//       they usually indicate that the "real value" you were looking
//       for does not exist. Instead, you have an error. The example
//       error union type of MyError!u8 means that foo either holds
//       a u8 value OR an error. There is _no value_ of type u8 in foo
//       when it's set to an error!
//
// * void
//
//       var foo: void = {};
//
//       "void" is a _type_, not a value. It is the most popular of the
//       Zero Bit Types (those types which take up absolutely no space
//       and have only a semantic value. When compiled to executable
//       code, zero bit types generate no code at all. The above example
//       shows a variable foo of type void which is assigned the value
//       of an empty expression. It's much more common to see void as
//       the return type of a function that returns nothing.
//
// Zig has all of these ways of expressing different types of "no value"
// because they each serve a purpose. Briefly:
//
//   * undefined - there is no value YET, this cannot be read YET
//   * null      - there is an explicit value of "no value"
//   * errors    - there is no value because something went wrong
//   * void      - there will NEVER be a value stored here
```

---

51. values (or how memory works for stuff)

This is a deep dive. These is not a killing house anymore. We are getting TO IT.

`@import()` that you use to import standard library for example is a way of telling the compiler to smash all of standard's library code together with your code to RAM while running the program. `const std` that keeps the import is just a struct.

Structs are built with fields (or methods too, but thats not important right now). Structs are not some abstract beings, if you add the memory taken by all struct's fields, you will know how much memory the struct takes.

Structs can be created in different ways.The narrator is created as a constant variable. The memory address of this structure will not change while the program runs, also none of the vields values will change, as it is initialized as a constant.

```zig
const the_narrator = Character{
    .gold = 12,
    .health = 99,
    .experience = 9000,
};
```

When you create a struct using `var`, you get a struct with still constant memory address, but the fields' values themselves can be changed.

```zig
var global_wizard{};
```

Functions on the other hand are kept as instruction codes at particular address. Function parameters are always immutable, and are stored in "the stack". The stack is a specific part of RAM that is reserved for the program you're running. CPU has special methods for adding/taking stuff from the stack, so it makes it really efficient. When the function executes, the parameters are often loaded to even faster memory -- CPU registers.

When we define a function, ie. `main` with no parameters, it will have a stack entry called "frame".

```zig
pub fn main() void {
    var glorp = Character{
    .gold = 30,
    };

    // according to ziglings note
    // glorp will be kept on the stack
    // "each instance of glorp is mutable and.."
    // "..therefore unique to the invocation of this fn"
    // it does not really sound like an explanation

    // ok i got it now, each time a function is called
    // ie by a new thread, a new frame is created on
    // te stack for it, in this frame a new instance of
    // glorp is created, so we avoid a situation in which
    // other caller changed some value of glorp (var -- mutable)
    // and the new caller would get the changed value

    const reward_xp: u32 = 200;

    // reward_xp on the other hand is constant, he does
    // not need to be copied and kept on stack, cause
    // no function call (thread, whatever) will change
    // him, its up to compiler where to put him -- either
    // in the global memory or inlince, FUSED into the
    // generated code

    const print = std.debug.print

    // as we said before, std is just some struct
    // debug is also a struct, just nested in std
    // print is a public function in the namespace of
    // that struct, we can just assign this function
    // to a new const
}
```

We will now look at different ways of assigning existing variables to new ones. When do we pass the same object in memory? When do we make a copy?

```zig
var glorp_access1: Character = glorp;
glorp_access1.gold = 111;
```

Above creates a copy. You can see it by changing a value after assigning to new name. The two variables will have different values if you change the value for one of them.

```zig
var glorp_access2: *Character = &glorp;
glorp_access2.gold = 222;
```

Now we created a proper link to an object. This variable is nothing more than just the original glorp in disguise. If you now change the value of some field, it will be changed for the original glorp as well.

```zig
const glorp_access3: *Character = &glorp;
glorp_access3.gold = 222;
```

Now we get back to the pointer differences we've talked when doing pointer exercises. `const` pointer variable means that we can't change what the pointer variable is pointing at, but we can still change the values of the variable we point at.

```zig
const glorp_access4: *const Character = &glorp;
glorp_access4 = 111;
```

Now this would result in an error. If we make the pointer type `consgt` as well, it means we only create a window that we can use to look at the original value. We can't do anything with it. It's like a zoo.

Last thing here: when arguments are passed to the function, they are always initialized as constant values. We can't reassign values to  function parameter variable.

```zig
fn func(arg: u8) void {
    arg = 42; // will result in an error
}
```

Really last thing. There are 3 types of memory allocated for programs: data segments (compile time, global and static stuff), stack (allocated at run time, fast, dynamic, but limited by system dependent memory size), and the heap (unlimited allocation up to RAM size, slower than stack, can lead to memory leaks if not handled properly).

52. (-53) slices

Slices are arrays of an undefined length. They are useful for example in functions, where we dont know what length the array we want to pass is. The type for slices instead of ie. `[10]u8` for 10 u8 elements is `[]u8` -- we define that it's an array and specify element types, but we don't tell what is the length. Slices also allow taking a... slice from an array. 

```zig
var arr = [3]u8{ 0, 1, 2 };
const foo = arr[0..1]; // 0
const boo = arr[0..]; // 0, 1, 2    
```

Under the hood slices are stored as first item and the length.

We can also manipulate strings, which after all are arrays too. IMMUTABLE arrays mind you. Which is why, we need to specify that in the type of slice `[]const u8`.

```zig
const scrambled = "some string hey";
const part1: []const u8 = scrambled[0..10];
```

---

53. manypointers

Dont't worry if you feel a little lost here. I was to, it's ok. Just read it a word at a time and rewrite in your own words whatever you don't understand.

Coercing is, usually automatic and implicit, conversion of one type to another. For example when you print an int in Python (I guess its converted from int to string, im not 100% but you get the gist). 

```zig
var foo: [4]u8: [4]u8{1, 2, 3, 4};
var foo_regular_ptr: *[4]u8 = &foo;
var foo_slice: []u8 = foo[0..];
var foo_ptr: [*]u8 = &foo;
var foo_slice_from_ptr: []u8 = foo_ptr[0..4];
```

You may now be thinking "what the fuck". Bear with me. Regular pointer (`*[5]u8`) can be used for arrays of known sizes, it's aware of the array length, therefore it's safe but less flexible. Manypointer (`[*]u8`) does not know about the length, more than that, this ignorance allows more flexibility, but you need to be cautious. Compiler will not throw an error when you try to access out-of-bounds elements.

```zig
var arr: [5]u8 = [_]u8{1, 2, 3, 4, 5};

// Regular array pointer
var regular_ptr: *[5]u8 = &arr;
_ = regular_ptr[4]; // Safe, bounds-checked
// _ = regular_ptr[5]; // This would be a compile-time error

// Many-item pointer
var many_ptr: [*]u8 = &arr;
_ = many_ptr[4]; // Works, but not bounds-checked
_ = many_ptr[5]; // This compiles but could lead to undefined behavior
```

Also, note that when we create `foo_slice_from_ptr`, we specify the length by `[0..4]`. It may result in accessing out of bounds memory, because we need to be in control of the array length. The pointer does not know the length, so we could write `&arr[0..5]` and it would pass compilation, but access memory it shouldn't access, cause `arr` has only 4 elements.

In principle the main difference between the regular pointer, manypointer, and slice is:
- regular: known length, forces you to adhere to this length and doesnt allow arbitrary lengths
- manypointer: unknown length, will do whatever you ask it to do, but you need to be in control of the correct index access, cause you can get out-of-bands
- slice: arbitrary length, you will be kept in bounds by compiler

It may help to think of these pointers/slices as structs, for example slice could be defined as:

```zig
const slice = struct {
    ptr: [*]T,  // manypointer to an array containing elements of some type T
    len: usize, // remember for lengths we use usize, its size is CPU dependent
}
```

Now getting to the exercise:

```zig
// this is a constant pointer to a 21 u8 characters long array
// it does not allow mutating the array -- *const
const zen12: *const [21]u8 = "Memory is a resource";
// it would also be valid to set the type to a slice
// this will not work in the exercise tho
// you would need to then get a pointer to its
// first element
const zen12: []const u8 = "Memory is a resource";

const zen_manyptr: [*]const u8 = zen12;
// ...

// we now create a slice out of it
// remember that slice demands some known size
// this is why we add the indexing, we have to 
// know the valid indexes tho, so we don't get
// out-of-bounds
const zen12_string: []const u8 = zen_manyptr[0..21]
```

Cheat sheet from the exercise:

```zig
//     FREE ZIG POINTER CHEATSHEET! (Using u8 as the example type.)
//   +---------------+----------------------------------------------+
//   |  u8           |  one u8                                      |
//   |  *u8          |  pointer to one u8                           |
//   |  [2]u8        |  two u8s                                     |
//   |  [*]u8        |  pointer to unknown number of u8s            |
//   |  [*]const u8  |  pointer to unknown number of immutable u8s  |
//   |  *[2]u8       |  pointer to an array of 2 u8s                |
//   |  *const [2]u8 |  pointer to an immutable array of 2 u8s      |
//   |  []u8         |  slice of u8s                                |
//   |  []const u8   |  slice of immutable u8s                      |
//   +---------------+----------------------------------------------+
```


