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

