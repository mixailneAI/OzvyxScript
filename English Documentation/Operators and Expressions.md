# Operators and Expressions

## Introduction

An expression is a piece of code that produces a value. An operator is a symbol or keyword that combines values into a new value.

```
let sum = 2 + 3
```

Here `2` and `3` are values, `+` is an operator, and `2 + 3` is an expression that produces `5`.

This chapter covers every operator in OzvyxScript, the rules of precedence, and the way expressions are evaluated.

## Arithmetic operators

OzvyxScript provides the following arithmetic operators.

| Operator | Meaning | Example |
|----------|---------|---------|
| `+` | addition | `2 + 3` gives `5` |
| `-` | subtraction | `7 - 2` gives `5` |
| `*` | multiplication | `3 * 4` gives `12` |
| `/` | division | `10 / 3` gives `3` |
| `%` | remainder | `10 % 3` gives `1` |

Arithmetic operators work on `Int`, on sized integer types, and on `Float`.

```
let a = 2 + 3
let b = 7 - 2
let c = 3 * 4
let d = 10 / 3
let e = 10 % 3
```

Division on integers truncates toward zero.

```
println(10 / 3)      // prints 3
println(-10 / 3)     // prints -3
println(10 / -3)     // prints -3
```

The remainder operator follows the same rule as division.

```
println(10 % 3)      // prints 1
println(-10 % 3)     // prints -1
println(10 % -3)     // prints 1
```

Division by zero on integers is a runtime panic.

```
let x = 10 / 0       // panic: integer division by zero
```

To avoid the panic, check the divisor first.

```
let divisor = 0
if divisor != 0 {
    println(10 / divisor)
} else {
    println("cannot divide by zero")
}
```

Floating-point division does not panic on zero. It produces `inf`, `-inf`, or `nan`, following IEEE 754.

```
println(1.0 / 0.0)     // prints inf
println(-1.0 / 0.0)    // prints -inf
println(0.0 / 0.0)     // prints NaN
```

The arithmetic operators are left-associative. `a - b - c` is `(a - b) - c`.

```
println(10 - 3 - 2)    // prints 5
```

The unary minus changes the sign of a value.

```
let x = 5
let y = -x             // y is -5
```

## Comparison operators

Comparison operators produce a `Bool`.

| Operator | Meaning |
|----------|---------|
| `==` | equal to |
| `!=` | not equal to |
| `<` | less than |
| `<=` | less than or equal to |
| `>` | greater than |
| `>=` | greater than or equal to |

```
let a = 5 == 5         // true
let b = 5 != 5         // false
let c = 3 < 7          // true
let d = 3 <= 3         // true
let e = 7 > 3          // true
let f = 7 >= 8         // false
```

Comparisons work on all primitive types.

```
let a = "apple" == "apple"       // true
let b = 'x' < 'y'                // true
let c = true > false             // true (false < true)
let d = 1.5 <= 2.0               // true
```

For types that define an ordering, such as tuples, the comparison is lexicographic.

```
let a = (1, 2) < (1, 3)          // true, because 2 < 3
let b = (2, 0) < (1, 999)        // false, because 2 > 1
```

Types that do not have a defined ordering, such as `List` or user structs, can still be compared with `==` and `!=` if they implement the equality trait. Ordering comparisons on such types are a compile-time error unless the trait is implemented.

## Logical operators

Logical operators work on `Bool`.

| Operator | Meaning |
|----------|---------|
| `and` | logical AND |
| `or` | logical OR |
| `not` | logical NOT |

```
let a = true and false      // false
let b = true or false       // true
let c = not true            // false
let d = not (true and true) // false
```

Both `and` and `or` short-circuit.

For `and`, if the left operand is `false`, the right operand is not evaluated.

For `or`, if the left operand is `true`, the right operand is not evaluated.

```
fn expensive() -> Bool {
    println("evaluating")
    true
}

let a = false and expensive()   // prints nothing, a is false
let b = true or expensive()     // prints nothing, b is true
let c = true and expensive()    // prints "evaluating", c is true
```

This behavior is useful when the right operand would fail if evaluated.

```
let list = List.new()

if not list.is_empty() and list[0] > 0 {
    println("first element is positive")
}
```

If `list` is empty, `list[0]` would panic. Short-circuit prevents the panic.

## The assignment operator

The `=` symbol is the assignment operator.

```
let mut x = 5
x = 10
```

Assignment produces `Unit`, not the assigned value. This differs from C and JavaScript, where assignment is an expression.

```
let mut a = 1
let b = (a = 2)     // error: assignment does not produce a value
```

This rule prevents a common class of bugs, such as writing `if x = 5` when `if x == 5` was intended.

## Compound assignment

OzvyxScript provides compound assignments for the common case of applying an operator to the same variable.

| Operator | Equivalent to |
|----------|---------------|
| `+=` | `x = x + y` |
| `-=` | `x = x - y` |
| `*=` | `x = x * y` |
| `/=` | `x = x / y` |
| `%=` | `x = x % y` |

```
let mut x = 10
x += 5           // x is now 15
x -= 3           // x is now 12
x *= 2           // x is now 24
x /= 4           // x is now 6
x %= 4           // x is now 2
```

The variable must be mutable. Using a compound assignment on an immutable binding is a compile-time error.

## Bitwise operators

Bitwise operators work on integers.

| Operator | Meaning |
|----------|---------|
| `&` | bitwise AND |
| `\|` | bitwise OR |
| `^` | bitwise XOR |
| `~` | bitwise NOT |
| `<<` | left shift |
| `>>` | right shift |

```
let a = 0b1100 & 0b1010     // 0b1000, which is 8
let b = 0b1100 | 0b1010     // 0b1110, which is 14
let c = 0b1100 ^ 0b1010     // 0b0110, which is 6
let d = ~0b1100             // flips all bits
let e = 1 << 4              // 16
let f = 16 >> 2             // 4
```

The shift operators take the number of positions to shift. Shifting by more than the width of the type is undefined behavior and produces a compile-time error when the shift amount is known at compile time.

```
let x: U32 = 1
let y = x << 33             // error: shift amount is out of range
```

Bitwise operators are used in low-level code: flags, masks, packed data structures.

## The range operator

The range operator produces a sequence of integers.

```
for i in 1..5 {
    println(i)
}
```

This prints `1`, `2`, `3`, `4`. The right endpoint is exclusive.

An inclusive range includes the right endpoint.

```
for i in 1..=5 {
    println(i)
}
```

This prints `1`, `2`, `3`, `4`, `5`.

Ranges are values and can be stored.

```
let r = 1..10
for i in r {
    println(i)
}
```

A range with no upper bound is open.

```
for i in 1.. {
    println(i)
    if i >= 5 { break }
}
```

This is an infinite range that must be terminated by `break`.

## The pipeline operator

The pipeline operator `|>` passes a value from left to right.

```
let result = 5 |> double |> add_one
```

This is equivalent to

```
let result = add_one(double(5))
```

The pipeline is used to write data transformations in the order in which they occur.

```
let result = users
    |> filter(fn(u) => u.age >= 18)
    |> map(fn(u) => u.name)
    |> sort()
    |> join(", ")
```

Without the pipeline, the same code becomes nested.

```
let result = join(sort(map(filter(users, fn(u) => u.age >= 18), fn(u) => u.name)), ", ")
```

The pipeline does not change semantics. It only changes how the code reads.

The pipeline has low precedence. It binds more tightly than assignment but less tightly than everything else.

```
let x = 5 |> double |> add_one     // parses as 5 |> (double |> add_one)? No.
```

Actually, `|>` is left-associative, so the expression above parses as `(5 |> double) |> add_one`. This is the desired behavior.

## Precedence

Precedence determines which operators bind more tightly. The following table lists operators from highest to lowest.

| Level | Operators | Associativity |
|-------|-----------|---------------|
| 1 | `()` `[]` `.` `?` `::` | left |
| 2 | unary `-` `not` `ref` `move` | right |
| 3 | `*` `/` `%` | left |
| 4 | `+` `-` | left |
| 5 | `<<` `>>` | left |
| 6 | `&` | left |
| 7 | `^` | left |
| 8 | `\|` | left |
| 9 | `<` `<=` `>` `>=` | left |
| 10 | `==` `!=` | left |
| 11 | `and` | left |
| 12 | `or` | left |
| 13 | `..` `..=` | none |
| 14 | `\|>` | left |
| 15 | `=` `+=` `-=` etc. | right |

When in doubt, use parentheses. The compiler never requires them for correctness, but readers benefit from explicit grouping.

```
let a = 2 + 3 * 4         // 14, because * binds tighter than +
let b = (2 + 3) * 4       // 20
let c = 2 + 3 > 4         // true, because + binds tighter than >
```

## Evaluation order

OzvyxScript evaluates expressions left to right, with two exceptions.

The right-hand side of `and` and `or` is evaluated only when needed.

The right-hand side of the assignment is evaluated before the assignment.

```
let mut x = 0
fn next() -> Int {
    x += 1
    x
}

let a = next() + next()   // a is 3
let b = next() * next()   // b is 20 (4 * 5)
```

The order of evaluation is guaranteed. This makes code predictable and removes a class of bugs found in languages with unspecified evaluation order.

## Type of an expression

Every expression has a type. The compiler determines the type from the operands and the operator.

```
2 + 3        // Int
2.0 + 3.0    // Float
"a" + "b"    // String
true and false   // Bool
2 < 3        // Bool
1..5         // Range[Int]
```

If the operands have incompatible types, the compiler reports an error.

```
let x = 2 + "hello"     // error: expected `Int`, found `String`
```

There are no implicit conversions. Adding a number to a string requires an explicit conversion.

```
let x = 2
let y = "hello"
let z = x.to_string() + y   // ok
```

## The `as` operator

The `as` operator converts a value from one type to another.

```
let x: I32 = 100
let y: I64 = x as I64
```

For numeric types, `as` performs a bit-level conversion. No range check is performed.

```
let big: I64 = 1000
let small = big as I8     // silently truncates
```

For enum variants and trait objects, `as` performs an upcast or downcast. This is covered in later chapters.

The `as` operator is also used to rename imports.

```
use std::io as stdio
```

This makes `stdio::read_line` available instead of `io::read_line`.

## The `?` operator

The `?` operator propagates errors from a `Result` or `Option`.

```
fn read_number() -> Result[Int, ParseError] {
    let line = io::read_line()?
    let n = line.to_int()?
    Ok(n)
}
```

Here, if either `io::read_line()` or `line.to_int()` fails, the failure is returned immediately from `read_number`. The `?` operator short-circuits.

The `?` operator works only inside a function whose return type is `Result` or `Option` with a compatible error type.

The full behavior of `?` is covered in the chapter on error handling.

## The `ref` operator

The `ref` operator takes a reference to a value.

```
let x = 5
let r = ref x
println(*r)         // prints 5
```

The `*` operator dereferences a reference.

References are covered in detail in the chapter on ownership and borrowing.

## The `move` operator

The `move` operator transfers ownership of a value into a closure.

```
let name = "Alice"
let greet = move || println("Hello, " + name)
greet()
```

After `move`, the closure owns `name`. The original binding is no longer usable.

Ownership is covered in the chapter on ownership and borrowing.

## The indexing operator

The indexing operator `[]` accesses an element of a collection.

```
let list = [10, 20, 30]
println(list[0])         // prints 10
println(list[1])         // prints 20
```

Indexing is zero-based. Accessing an index out of range is a runtime panic.

```
let list = [10, 20, 30]
println(list[5])         // panic: index out of range
```

A safe alternative is `get`, which returns an `Option`.

```
let list = [10, 20, 30]
match list.get(5) {
    when Some(v) => println(v),
    when None    => println("out of range"),
}
```

## The method call operator

The `.` operator is used to call methods on a value.

```
let s = "Hello"
println(s.len())
println(s.to_uppercase())
```

Methods are functions attached to a type. They are covered in the chapter on structs and methods.

## The path operator

The `::` operator accesses a name inside a module or a type.

```
use std::io
let line = io::read_line()?

enum Color { Red, Green, Blue }
let c = Color::Red
```

The path operator is used for module-qualified names and for enum variants.

## Examples

### Computing an average

```
let values = [4, 8, 15, 16, 23, 42]
let sum = values.iter().fold(0, fn(a, b) => a + b)
let avg = sum as Float / values.len() as Float
println(avg)         // prints approximately 18.0
```

### Checking a password

```
fn is_strong(password: String) -> Bool {
    password.len() >= 12
    and password.chars().any(fn(c) => c.is_digit())
    and password.chars().any(fn(c) => c.is_uppercase())
    and password.chars().any(fn(c) => c.is_lowercase())
}
```

The `and` chain short-circuits: if the length is less than 12, the remaining checks are skipped.

### FizzBuzz with the pipeline

```
fn classify(n: Int) -> String {
    match (n % 3, n % 5) {
        when (0, 0) => "FizzBuzz",
        when (0, _) => "Fizz",
        when (_, 0) => "Buzz",
        when _     => n.to_string(),
    }
}

1..=20
    |> map(classify)
    |> join("\n")
    |> println()
```

### Parsing a comma-separated list

```
fn parse_list(input: String) -> List[Int] {
    input
        |> split(",")
        |> map(fn(s) => s.trim())
        |> filter(fn(s) => s.len() > 0)
        |> map(fn(s) => s.to_int().unwrap_or(0))
}
```

## Common mistakes

Using `=` when `==` is intended.

```
if x = 5 { ... }       // error: assignment does not produce a value
if x == 5 { ... }      // ok
```

Assuming that `+` works on mixed types.

```
let x = 2 + 2.0        // error: mismatched types
let x = 2.0 + 2.0      // ok
```

Forgetting that integer division truncates.

```
let x = 7 / 2          // 3, not 3.5
let x = 7.0 / 2.0      // 3.5
```

Using `not` as a method or a symbol.

```
let x = !true          // error: `!` is not an operator in OzvyxScript
let x = not true       // ok
```

Mixing `and`/`or` with `&&`/`||`.

```
let x = true && false  // error: `&&` is not an operator in OzvyxScript
let x = true and false // ok
```

Forgetting that ranges are exclusive at the upper bound by default.

```
for i in 0..10 { println(i) }    // prints 0 through 9
for i in 0..=10 { println(i) }   // prints 0 through 10
```

Assuming that `?` can be used anywhere.

```
fn f() -> Int {
    let x = g()?       // error: `?` requires a Result or Option return type
    x
}
```

Fix:

```
fn f() -> Result[Int, Error] {
    let x = g()?
    Ok(x)
}
```

## Summary

- Arithmetic operators work on integers and floats. Integer division truncates toward zero.
- Comparison operators produce `Bool` and work on all primitive types and tuples.
- Logical operators `and`, `or`, `not` work on `Bool` and short-circuit.
- Assignment produces `Unit`, not a value. This prevents a common class of bugs.
- Compound assignments `+=`, `-=`, and so on, are shortcuts for common patterns.
- Bitwise operators work on integers and are used in low-level code.
- The range operators `..` and `..=` produce sequences of integers.
- The pipeline operator `|>` passes a value from left to right, improving readability.
- Precedence follows a fixed table. Use parentheses when in doubt.
- Evaluation order is left to right, with short-circuiting for `and` and `or`.
- The `?` operator propagates errors from `Result` and `Option`.
- The `ref`, `move`, and `*` operators relate to ownership and are covered later.

## Next

The next chapter, **Control Flow**, covers `if`, `else`, and the way conditions and branching are expressed in OzvyxScript.
