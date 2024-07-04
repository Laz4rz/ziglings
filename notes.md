I tried to install neovim while doing these, but I failed (.local permissions problems) -- vim ftw.

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

4. 
