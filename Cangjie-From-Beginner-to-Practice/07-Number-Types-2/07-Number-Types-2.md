# 07-Numeric Types (2)

## Integer Type Literals

**Integer literals are code that can be visually identified as integers**

For example:

```
100 integer
200 integer
xdfds not an integer
```

Integer type literals have 4 different base representations: binary (using `0b` or `0B` prefix), octal (using `0o` or `0O` prefix), decimal (no prefix), and hexadecimal (using `0x` or `0X` prefix). For example, the decimal number `24` is represented as `0b00011000` (or `0B00011000`) in binary, `0o30` (or `0O30`) in octal, and `0x18` (or `0X18`) in hexadecimal. Numbers are decimal by default. For example, `100` is generally understood as `100` in decimal.

In various base representations, the underscore `_` can be used as a separator to facilitate the identification of the number of digits, such as `0b0001_1000`.

For integer type literals, if their value exceeds the representation range of the integer type required by the context, the compiler will report an error.

```typescript
let x: Int8 = 128; // Error, 128 out of the range of Int8
let y: UInt8 = 256; // Error, 256 out of the range of UInt8
let z: Int32 = 0x8000_0000; // Error, 0x8000_0000 out of the range of Int32
```

When using integer type literals, you can add a **suffix** to explicitly specify the type of the integer literal. The correspondence between suffixes and types is as follows:

| Suffix | Type  | Suffix | Type   |
| :--- | :---- | :--- | :----- |
| i8   | Int8  | u8   | UInt8  |
| i16  | Int16 | u16  | UInt16 |
| i32  | Int32 | u32  | UInt32 |
| i64  | Int64 | u64  | UInt64 |

Integer literals with suffixes can be used in the following way:

```typescript
var x = 100i8  // x is 100 with type Int8
var y = 0x10u64 // y is 16 with type UInt64
var z = 0o432i32  // z is 282 with type Int32
```

## Character Byte Literals

Cangjie programming language supports character byte literals to facilitate the use of ASCII codes to represent values of type `UInt8`. A character byte literal consists of the character b, a pair of single quotes marking the beginning and end, and an `ASCII` character, for example:

```typescript
var a = b'x' // a is 120 with type UInt8
var b = b'\n' // b is 10 with type UInt8
var c = b'\u{78}' // c is 120 with type UInt8
```

`b'x'` represents a literal value of type UInt8 with a size of 120. Additionally, `b'\u{78}'` can be used as an escape form to represent a literal value of type `UInt8` with a hexadecimal size of 0x78 or a decimal size of 120. Note that `\u` can contain at most two hexadecimal digits internally, and the value must be less than 256 (decimal).

### ASCII Table

| Decimal | Hexadecimal | Character |
| ------ | -------- | ---- |
| 48     | 30       | 0    |
| 49     | 31       | 1    |
| 50     | 32       | 2    |
| 51     | 33       | 3    |
| 52     | 34       | 4    |
| 53     | 35       | 5    |
| 54     | 36       | 6    |
| 55     | 37       | 7    |
| 56     | 38       | 8    |
| 57     | 39       | 9    |
| 65     | 41       | A    |
| 66     | 42       | B    |
| 67     | 43       | C    |
| 68     | 44       | D    |
| 69     | 45       | E    |
| 70     | 46       | F    |
| 71     | 47       | G    |
| 72     | 48       | H    |
| 73     | 49       | I    |
| 74     | 4A       | J    |
| 75     | 4B       | K    |
| 76     | 4C       | L    |
| 77     | 4D       | M    |
| 78     | 4E       | N    |
| 79     | 4F       | O    |
| 80     | 50       | P    |
| 81     | 51       | Q    |
| 82     | 52       | R    |
| 83     | 53       | S    |
| 84     | 54       | T    |
| 85     | 55       | U    |
| 86     | 56       | V    |
| 87     | 57       | W    |
| 88     | 58       | X    |
| 89     | 59       | Y    |
| 90     | 5A       | Z    |
| 97     | 61       | a    |
| 98     | 62       | b    |
| 99     | 63       | c    |
| 100    | 64       | d    |
| 101    | 65       | e    |