# 08-Floating-Point Types (1)

Floating-point types include `Float16`, `Float32`, and `Float64`, which are used to represent floating-point numbers (numbers with decimal parts, such as 3.14159, 8.24, and 0.1) with encoding lengths of `16-bit`, `32-bit`, and `64-bit` respectively. `Float16`, `Float32`, and `Float64` correspond to half-precision format (binary16), single-precision format (binary32), and double-precision format (binary64) in IEEE 754.

`Float64` has a precision of approximately 15 decimal places, `Float32` has a precision of approximately 6 decimal places, and `Float16` has a precision of approximately 3 decimal places. Which floating-point type to use depends on the nature and range of the floating-point numbers that need to be processed in the code. **In cases where multiple floating-point types are suitable, higher precision floating-point types are preferred, because the cumulative calculation errors of lower precision floating-point types can easily spread, and the range of integers they can accurately represent is also very limited**.

## Floating-Point Type Literals

Floating-point type literals have two base representations: decimal and hexadecimal. In decimal representation, a floating-point literal must include at least one integer part or one decimal part. When there is no decimal part, it must include an exponent part (prefixed with `e` or `E`, with a base of 10). In hexadecimal representation, a floating-point literal must include at least one integer part or decimal part (prefixed with `0x` or `0X`), and must also include an exponent part (prefixed with `p` or `P`, with a base of 2).

The following examples demonstrate the use of floating-point literals:

```javascript
let a: Float32 = 3.14
let b: Float32 = 2e3
let c: Float32 = 2.4e-1
let d: Float64 = .123e2
let e: Float64 = 0x1.1p0
let f: Float64 = 0x1p2
let g: Float64 = 0x.2p4
```

When using decimal floating-point literals, you can add a suffix to explicitly specify the type of the floating-point literal. The correspondence between suffixes and types is as follows:

## Suffix Representation for Floating-Point Literals

| Suffix | Type    |
| :--- | :------ |
| f16  | Float16 |
| f32  | Float32 |
| f64  | Float64 |

Floating-point literals with suffixes can be used in the following way:

```javascript
let a = 3.14f32   // a is 3.14 with type Float32
let b = 2e3f32    // b is 2e3 with type Float32
let c = 2.4e-1f64 // c is 2.4e-1 with type Float64
let d = .123e2f64 // d is .123e2 with type Float64
```