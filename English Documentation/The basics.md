
# The Basics

## What OzvyxScript is

OzvyxScript is a statically typed programming language designed for systems programming, application development, and scripting. It compiles to native code, does not use a garbage collector by default, and provides live error diagnostics through an editor integration called the Language Server Protocol.

Source files use the extension `.yx` and are always encoded in UTF-8.

The compiler is called `ozvk`. It also serves as the project manager, package manager, REPL, formatter, test runner, and documentation generator.

## Your first program

Create a file named `hello.yx` and write a single line:

```
println("Hello, World!")
```

Run it:

```
ozvk run hello.yx
```

The output is:

```
Hello, World!
```

That is a complete OzvyxScript program. No imports, no function declaration, no entry point annotation. Top-level code is the implicit entry point of the program.

## Program structure

An OzvyxScript file has two possible shapes.

The first is a script: top-level code that runs immediately. This is the shape used for small programs, experiments, and one-off tools.

```
let name = "World"
println("Hello, " + name)
```

The second is a module: a collection of declarations. This shape is used when the file is part of a larger project.

```
fn greet(name: String) {
    println("Hello, " + name)
}

fn main() {
    greet("World")
}
```

A file cannot mix both shapes. If a file has a `fn main`, top-level code is not allowed. If a file has top-level code, `fn main` must not be declared.

The compiler reports this conflict as error `OZX0201`.

## Comments

OzvyxScript has three kinds of comments.

Line comments begin with `//` and run to the end of the line.

```
// This is a line comment.
let x = 5
```

Block comments begin with `/*` and end with `*/`. They can span multiple lines.

```
/* This is
   a block comment. */
let y = 10
```

Documentation comments begin with `///` and attach to the declaration that follows them.

```
/// Returns the sum of two integers.
fn add(a: Int, b: Int) -> Int {
    a + b
}
```

Documentation comments are collected by the compiler and can be rendered as HTML with `ozvk doc`.

## Statements

A statement is a single unit of action. In OzvyxScript, statements are separated by newlines. A semicolon `;` may be used as an optional separator when two statements appear on the same line.

```
let a = 1
let b = 2
let c = 3
```

The same three statements on one line:

```
let a = 1; let b = 2; let c = 3
```

The semicolon is never required. Its only purpose is to allow multiple statements on one line, which is useful in short REPL sessions and in compact test cases.

## Variables

Variables are declared with `let`.

```
let x = 5
let name = "Alice"
let is_ready = true
```

A variable declared with `let` is immutable. Attempting to reassign it is a compile-time error.

```
let x = 5
x = 10            // error: cannot assign to immutable variable `x`
```

To declare a mutable variable, use `let mut`.

```
let mut count = 0
count = count + 1
count = count + 1
println(count)    // prints 2
```

Immutability is the default because most values in a program do not need to change. Making mutation explicit helps the reader understand where state can be modified.

## Types

Every value in OzvyxScript has a type. The compiler infers types automatically in most cases, but you can also write them explicitly.

```
let x = 5                 // x is Int
let y: Int = 5            // same thing, explicit
let name = "Alice"        // name is String
let pi = 3.14             // pi is Float
let ready = true          // ready is Bool
```

The basic types are:

- `Int` — a 64-bit signed integer.
- `Float` — a 64-bit floating-point number.
- `Bool` — either `true` or `false`.
- `Char` — a single Unicode character.
- `String` — a sequence of UTF-8 characters.
- `Unit` — the type of expressions that produce no value.

Type annotations are optional when the compiler can infer the type from the value. They are required when the value alone does not determine the type, or when you want to document intent explicitly.

## Printing output

The simplest way to print is `println`, which writes a line to standard output.

```
println("Hello")
println(42)
println(true)
```

To print without a trailing newline, use `print`.

```
print("Loading")
print(".")
print(".")
print(".\n")
```

Output:

```
Loading...
```

## Reading input

To read a line of text from standard input, use `io::read_line`.

```
use std::io

let name = io::read_line()
println("Hello, " + name)
```

The `use` keyword imports a module. The path `std::io` refers to the `io` module of the standard library.

`io::read_line` returns a `Result` because reading input can fail. For now, assume it succeeds. Error handling is covered in a later chapter.

## A first complete program

Combining everything so far, here is a program that asks for a name and greets the user.

```
use std::io

print("What is your name? ")
let name = io::read_line()?
println("Hello, " + name + "!")
```

Run it:

```
$ ozvk run greet.yx
What is your name? Alice
Hello, Alice!
```

The `?` after `io::read_line()` propagates errors upward. If reading fails, the program exits with a diagnostic. In the next chapters you will learn how to handle errors explicitly.

## Expressions and values

In OzvyxScript, almost everything is an expression. An expression produces a value.

```
let x = if true { 1 } else { 2 }
```

Here `if true { 1 } else { 2 }` is an expression that evaluates to `1`. It is assigned to `x`.

The same is true for `match`, blocks, and function calls.

```
let result = {
    let a = 10
    let b = 20
    a + b
}
```

The block produces the value `30` because the last expression in the block is `a + b`. Note there is no `return` keyword and no trailing semicolon: the value of the block is the value of its last expression.

## Blocks

A block is a sequence of statements enclosed in braces `{ }`. Blocks introduce a new scope.

```
let x = 10

{
    let x = 20
    println(x)    // prints 20
}

println(x)        // prints 10
```

The inner `x` is a different variable from the outer `x`. The inner one goes out of scope at the closing brace.

Blocks are expressions, so they can be assigned or returned.

```
let value = {
    let a = 5
    let b = 7
    a * b
}
println(value)    // prints 35
```

## Naming conventions

OzvyxScript follows a consistent set of naming rules.

- Variable names use `snake_case`: `user_name`, `total_count`.
- Function names use `snake_case`: `read_file`, `calculate_sum`.
- Type names use `PascalCase`: `HttpClient`, `Point`.
- Constants use `UPPER_SNAKE_CASE`: `MAX_RETRIES`, `DEFAULT_PORT`.
- Module names use `snake_case`: `http_client`, `json_parser`.

The compiler emits a warning if a name does not follow the convention. The warning can be silenced with `#[allow(naming)]` when there is a reason.

## Formatting

The formatter is built into `ozvk`. Running

```
ozvk fmt
```

rewrites every `.yx` file in the current project to follow the standard style. The style is fixed and not configurable. It uses four spaces for indentation, a space around binary operators, and braces on the same line as the declaration.

Running `ozvk fmt` is recommended before every commit. Many editors can run it automatically on save.

## Running and building

To run a file directly:

```
ozvk run hello.yx
```

To build an executable:

```
ozvk build hello.yx
```

The output is placed in `target/debug/hello` by default. For an optimized build:

```
ozvk build --release hello.yx
```

The optimized build goes to `target/release/hello`.

To type-check without producing an executable:

```
ozvk check hello.yx
```

This is faster than a full build and is useful in continuous integration.

## The REPL

The interactive shell is started with

```
ozvk repl
```

Inside the REPL, you can type expressions and see their results immediately.

```
$ ozvk repl
OzvyxKompilator v1.0.0
>>> let x = 5
>>> x * 2
10
>>> println("Hello")
Hello
>>> :type x
Int
>>> :quit
```

The REPL keeps state between lines, so variables declared on one line are available on the next.

## Errors

OzvyxScript reports errors with a stable code, an exact location, a message, a suggestion, and a link to the documentation.

```
error[OZX0301]: expected `Int`, found `String`
  --> main.yx:3:18
   |
 3 |     let x: Int = "hello"
   |                  ^^^^^^^
   = help: convert the string to a number with `"hello".to_int()?`
   = docs: https://ozvyx.dev/errors/OZX0301
```

Every error code can be looked up on the documentation site. The same diagnostics appear in the editor when the Language Server Protocol integration is active.

## Common first mistakes

Forgetting to use `mut` before reassigning:

```
let x = 5
x = 10            // error: cannot assign to immutable variable
```

Fix:

```
let mut x = 5
x = 10            // ok
```

Using `=` instead of `==` for comparison:

```
if x = 5 { ... }  // error: expected `Bool`, found assignment
```

Fix:

```
if x == 5 { ... }
```

Forgetting parentheses around the condition is fine — the language does not require them. But if you come from C or Java, you may write them out of habit.

```
if (x == 5) { ... }   // ok, but unnecessary
if x == 5 { ... }     // preferred
```

Mixing top-level code with `fn main`:

```
fn main() {
    println("Hello")
}

println("World")     // error: top-level code together with `fn main`
```

Choose one or the other.

## Summary

- A `.yx` file is either a script (top-level code) or a module (declarations).
- Variables are declared with `let` and are immutable by default. Use `let mut` for mutable variables.
- Statements are separated by newlines. Semicolons are optional.
- Comments use `//`, `/* */`, and `///` for documentation.
- Blocks are expressions. The value of a block is its last expression.
- Names follow fixed conventions: `snake_case` for values, `PascalCase` for types.
- `ozvk run`, `ozvk build`, `ozvk check`, `ozvk fmt`, and `ozvk repl` are the primary commands.
- Errors carry a stable code, a location, a suggestion, and a documentation link.

## Next

The next chapter, **Variables and Types**, covers the type system in detail: integers, floating-point numbers, booleans, characters, strings, tuples, and how the compiler infers types.
