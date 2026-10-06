# Variables and Types

## Introduction

This chapter covers how values are stored, named, and typed in OzvyxScript. It explains every primitive type, the rules of type inference, the difference between mutable and immutable bindings, and the way the compiler decides what type an expression has.

By the end of the chapter you will be able to read and write type annotations confidently, and to understand why the compiler accepts or rejects a given piece of code.

## Bindings

A binding associates a name with a value.

```
let x = 5
```

Here `x` is a binding, `5` is a value, and `Int` is the type of the value. The compiler infers the type from the value, so no annotation is required.

A binding lives until the end of the block in which it was introduced. After that, the name is no longer available.

```
{
    let x = 5
    println(x)     // ok
}

println(x)         // error: `x` is not defined
```

Bindings may shadow each other. Shadowing creates a new binding that hides the previous one.

```
let x = 5
let x = "hello"
println(x)         // prints hello
```

The second `let x` does not modify the first. It creates a fresh binding. The first `x` is still alive but is no longer reachable by name.

Shadowing is often used to change the type of a value while keeping a meaningful name.

```
let input = "42"
let input = input.to_int()?
println(input + 1)     // prints 43
```

## Mutability

Bindings are immutable by default. Reassigning an immutable binding is a compile-time error.

```
let x = 5
x = 10              // error: cannot assign to immutable variable `x`
```

To allow reassignment, add `mut`.

```
let mut x = 5
x = 10              // ok
x = x + 1           // ok
println(x)          // prints 11
```

Mutability is a property of the binding, not of the value. Two bindings may point to the same value, one mutable and one not.

```
let a = 5
let mut b = a
b = 10
println(a)          // prints 5
println(b)          // prints 10
```

The assignment `b = 10` changes the binding `b`, not the value `a`. For primitive types like `Int`, each binding holds its own copy.

Immutable bindings are the default because most values in a program do not change after they are created. Marking mutability explicitly makes the places where state can change visible at a glance.

## Type annotations

A binding may include an explicit type.

```
let x: Int = 5
let name: String = "Alice"
let ready: Bool = true
```

The annotation is written after the name, separated by a colon.

Type annotations are optional when the compiler can infer the type from the value. They are required in three situations:

When the value alone does not determine the type.

```
let empty: List[Int] = List.new()
```

When you want to constrain a numeric literal.

```
let x: Float = 5
```

Without the annotation, `5` would be an `Int`.

When the annotation documents intent and improves readability.

```
let timeout: Duration = 30.seconds()
```

The compiler never requires an annotation for a value whose type is obvious. It never rejects an annotation for a value whose type is compatible.

## The primitive types

OzvyxScript has six primitive types.

### Int

`Int` is a 64-bit signed integer. Its range is from `-9_223_372_036_854_775_808` to `9_223_372_036_854_775_807`.

```
let a = 42
let b = -100
let c = 1_000_000
```

Underscores may appear anywhere inside a numeric literal to improve readability. They are ignored by the compiler.

Integer literals may be written in hexadecimal, octal, or binary.

```
let hex = 0xFF
let oct = 0o755
let bin = 0b1010
```

The type of an integer literal is `Int` unless the context requires otherwise.

### Sized integers

When a specific width is needed, use one of the explicitly sized integer types.

```
let a: I8 = -128
let b: U8 = 255
let c: I32 = 2_000_000
let d: U64 = 18_446_744_073_709_551_615
```

The available types are `I8`, `I16`, `I32`, `I64`, `I128` for signed integers, and `U8`, `U16`, `U32`, `U64`, `U128` for unsigned integers.

Sized types are used in low-level code, when interfacing with C libraries, and when working with binary data.

### Float

`Float` is a 64-bit IEEE 754 floating-point number.

```
let pi = 3.14159
let e = 2.71828
let neg = -1.5
let sci = 1.5e10
```

The type of a floating-point literal is `Float`.

A 32-bit variant is available when needed.

```
let x: F32 = 3.14
```

### Bool

`Bool` has two values: `true` and `false`.

```
let ready = true
let done = false
```

The result of any comparison is `Bool`.

```
let positive = 5 > 0        // true
let equal = 5 == 5          // true
```

Logical operators also produce `Bool`.

```
let both = true and false   // false
let either = true or false  // true
let negated = not true      // false
```

### Char

`Char` is a single Unicode scalar value, written inside single quotes.

```
let a = 'a'
let digit = '7'
let cyrillic = 'Ж'
let emoji = '🌌'
```

A `Char` occupies four bytes in memory. It may represent any code point from `U+0000` to `U+10FFFF`, except for the surrogate range.

Common escape sequences are available.

```
let newline = '\n'
let tab = '\t'
let backslash = '\\'
let quote = '\''
let unicode = '\u{1F30C}'
```

### String

`String` is a sequence of UTF-8 encoded characters.

```
let greeting = "Hello, World"
let empty = ""
let multiline = "line one\nline two"
```

Strings are discussed in detail in a later chapter. For now, know that they can be concatenated with `+` and compared with `==`.

```
let a = "Hello"
let b = "World"
let both = a + ", " + b + "!"
println(both)                 // prints Hello, World!
```

### Unit

`Unit` is the type of an expression that produces no useful value. It has exactly one value, written `()`.

```
let nothing = ()
```

Functions that do not return a value have return type `Unit`, which is usually omitted from the signature.

```
fn greet(name: String) {
    println("Hello, " + name)
}
```

The return type of `greet` is `Unit`.

## Type inference

The compiler infers the type of most expressions from their context.

```
let x = 5                // Int
let y = 5 + 3            // Int
let z = 5.0 + 3.0        // Float
let name = "Alice"       // String
let ready = x > y        // Bool
```

When the compiler cannot infer a type, it reports an error.

```
let x = List.new()       // error: cannot infer type of `x`
```

The annotation resolves the ambiguity.

```
let x: List[Int] = List.new()
```

Type inference in OzvyxScript is based on the Hindley–Milner algorithm with bidirectional checking. In practice this means the compiler can infer the type of complex expressions without annotations, but it never guesses. If the type is genuinely ambiguous, it asks for help.

## Numeric conversions

OzvyxScript does not perform implicit conversions between numeric types. Every conversion must be explicit.

```
let a: I32 = 10
let b: I64 = a           // error: expected `I64`, found `I32`
```

To convert, use `as`.

```
let a: I32 = 10
let b: I64 = a as I64    // ok
```

The `as` operator performs a bit-level conversion. It does not check whether the value fits in the target type. Converting a large `I64` to `I8` silently truncates.

```
let big: I64 = 1000
let small = big as I8    // silent truncation
```

To check whether a conversion is safe, use a checked conversion from the standard library.

```
let big: I64 = 1000
let small = big.to_i8()?   // returns Result[I8, ConvertError]
```

Implicit conversion between `Int` and `Float` is also forbidden.

```
let a: Int = 5
let b: Float = a         // error: expected `Float`, found `Int`
```

Explicit conversion:

```
let a: Int = 5
let b: Float = a as Float
```

This rule prevents a large class of subtle bugs. The cost is a few extra characters, and the benefit is a program whose numeric behavior is fully predictable.

## Tuples

A tuple groups several values of different types into one.

```
let point = (3, 4)
let person = ("Alice", 30, true)
```

The type of a tuple is written as the list of its element types.

```
let point: (Int, Int) = (3, 4)
let person: (String, Int, Bool) = ("Alice", 30, true)
```

Tuples are accessed by index, starting from zero.

```
let point = (3, 4)
println(point.0)     // prints 3
println(point.1)     // prints 4
```

Tuples are usually destructured with a pattern.

```
let (x, y) = (3, 4)
println(x + y)       // prints 7
```

A one-element tuple is not allowed. A tuple must have at least two elements, or zero elements.

The zero-element tuple is the value of `Unit`.

```
let nothing: () = ()
```

Tuples are useful for returning multiple values from a function.

```
fn min_max(values: List[Int]) -> (Int, Int) {
    let mut lo = values[0]
    let mut hi = values[0]
    for v in values {
        if v < lo { lo = v }
        if v > hi { hi = v }
    }
    (lo, hi)
}
```

## Type aliases

A type alias gives a new name to an existing type.

```
type UserId = U64
type Point = (Int, Int)
type Result = Result[String, IoError]
```

A type alias does not create a new type. It is only a name. Values of type `UserId` and `U64` are interchangeable.

```
type UserId = U64

fn find_user(id: UserId) -> Option[User] { ... }

let n: U64 = 42
find_user(n)     // ok
```

Type aliases are used for documentation and readability, not for safety. When a distinct type is needed, use a `struct` (covered in a later chapter).

## Constants

A constant is a value known at compile time.

```
const MAX_RETRIES: Int = 3
const PI: Float = 3.14159
const APP_NAME: String = "Ozvyx"
```

Constants must have an explicit type. They cannot be reassigned.

Constants are written in `UPPER_SNAKE_CASE` by convention.

## Naming rules

A name in OzvyxScript:

- Begins with a letter or an underscore.
- Continues with letters, digits, or underscores.
- Is case-sensitive.
- Cannot be a keyword.

The compiler follows the naming conventions listed in the first chapter. It emits warnings for violations and allows them to be silenced with an attribute.

```
#[allow(naming)]
let HTTP_PORT = 8080
```

## The `_` placeholder

The underscore `_` is a special name that discards a value.

```
let _ = some_expensive_computation()
```

The value is computed but not stored. The underscore may be used in bindings and in patterns.

```
let (_, y) = (3, 4)     // only y is bound
```

A binding whose name begins with an underscore is not reported as unused.

```
let _unused = 5
```

This is useful when a name is required by syntax but not needed in practice.

## Type annotations on expressions

A type annotation may be attached to any expression, not only to a binding.

```
let x = (5: Int)
let y = (if true { 1 } else { 2 }: Int)
```

This is rarely necessary. It is useful when the compiler needs a hint to resolve an ambiguous literal.

```
let x = (0: U8)
```

Without the annotation, `0` would be an `Int`.

## Example: unit conversion

The following program reads a number in meters and prints it in feet, inches, and miles. It uses several primitive types and conversions.

```
use std::io

fn meters_to_feet(m: Float) -> Float {
    m * 3.28084
}

fn meters_to_inches(m: Float) -> Float {
    m * 39.3701
}

fn meters_to_miles(m: Float) -> Float {
    m / 1609.344
}

print("Distance in meters: ")
let input = io::read_line()?
let meters: Float = input.to_float()?

println("Feet:   " + meters_to_feet(meters).to_string())
println("Inches: " + meters_to_inches(meters).to_string())
println("Miles:  " + meters_to_miles(meters).to_string())
```

The explicit annotation `let meters: Float` is required because `to_float` returns `Result[Float, ParseError]`, and the `?` unwraps it. The type of the unwrapped value is `Float`, and the annotation makes this clear to the reader.

## Common mistakes

Forgetting `mut` and trying to reassign.

```
let count = 0
count = count + 1        // error: cannot assign to immutable variable
```

Fix:

```
let mut count = 0
count = count + 1
```

Mixing signed and unsigned integers without conversion.

```
let a: I32 = -1
let b: U32 = 1
let c = a + b            // error: mismatched types
```

Fix:

```
let a: I32 = -1
let b: U32 = 1
let c = a as I64 + b as I64
```

Forgetting that integer division truncates.

```
let x = 5 / 2
println(x)               // prints 2, not 2.5
```

Fix:

```
let x = 5.0 / 2.0
println(x)               // prints 2.5
```

Assuming that a string can be added to a number.

```
let x = 5
let name = "count: " + x   // error: mismatched types
```

Fix:

```
let x = 5
let name = "count: " + x.to_string()
```

## Summary

- Bindings are declared with `let` and are immutable by default. Add `mut` to allow reassignment.
- Type annotations are optional when the compiler can infer the type.
- The primitive types are `Int`, `Float`, `Bool`, `Char`, `String`, and `Unit`.
- Sized integer types are available when a specific width is needed.
- No implicit numeric conversions. Use `as` or a checked conversion.
- Tuples group values of different types. They are accessed by index or destructured.
- Type aliases give a new name to an existing type.
- Constants are compile-time values with `UPPER_SNAKE_CASE` names.
- Shadowing allows a name to be reused with a different type.
- The underscore `_` discards a value and silences unused-variable warnings.

## Next

The next chapter, **Operators and Expressions**, covers arithmetic, comparison, logic, the pipeline operator, and the rules of precedence and associativity.
