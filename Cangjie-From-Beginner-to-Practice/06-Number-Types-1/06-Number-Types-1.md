# 06-Number Types (1)

## What are Data Types

In the world of programming, many times the targets we need to operate on are data. For example, warehouse management systems need to use data to register various equipment information, and flight management systems use data to record flight information. To facilitate the management of various types of data, we can design different data types to make it convenient for our programs to operate. In the Cangjie programming language, there are a total of 10 **basic data types**. They are:

| Data Type      | Brief Description                                                   |
| -------------- | ------------------------------------------------------------------- |
| Integer Type   | Used to represent integer values.                                   |
| Floating Type  | Used to represent numerical values with decimal parts.              |
| Boolean Type   | Stores logical values representing true or false.                   |
| Character Type | Represents a single character.                                      |
| String Type    | Can store text content composed of multiple characters.             |
| Tuple Type     | Combines multiple data of different types together.                 |
| Array Type     | Can store a group of data elements of the same type.                |
| Range Type     | Can represent numerical ranges within a certain scope.              |
| Unit Type      | Often represents specific semantic situations like no return value. |
| Nothing Type   | Represents the concept of empty or nothing.                         |

## Integer Types

Integer types are divided into signed integer types and unsigned (non-negative) integer types.

**Signed integer types** include `Int8`, `Int16`, `Int32`, `Int64`, and `IntNative`, which are used to represent types of signed integer values with encoding lengths of `8-bit`, `16-bit`, `32-bit`, `64-bit`, and platform-dependent size respectively.

**Unsigned integer types** include `UInt8`, `UInt16`, `UInt32`, `UInt64`, and `UIntNative`, which are used to represent types of unsigned integer values with encoding lengths of `8-bit`, `16-bit`, `32-bit`, `64-bit`, and platform-dependent size respectively.

For signed integer types with encoding length `N`, the representation range is: `-2^(N-1) ~ 2^(N-1) - 1`; for unsigned integer types with encoding length `N`, the representation range is: `0~2^(N-1)`. The following table lists the representation ranges of all integer types:

![image-20241216073728167](06-%E6%95%B0%E5%AD%97%E7%B1%BB%E5%9E%8B%EF%BC%881%EF%BC%89.assets/image-20241216073728167.png)

Which integer type a program specifically uses depends on the nature and range of integers that need to be processed in that program. When the `Int64` type is suitable, the `Int64` type is preferred because `Int64` has a sufficiently large representation range, and integer type literals are inferred as `Int64` type by default when there is no type context, which can avoid unnecessary type conversions.

## Integer Type Literals

**Integer literals are code that can be recognized as integers at a glance**

For example:

```
100 integer
200 integer
xdfds not an integer
```

Integer type literals have 4 base representation forms: binary (using `0b` or `0B` prefix), octal (using `0o` or `0O` prefix), decimal (no prefix), hexadecimal (using `0x` or `0X` prefix). For example, for the decimal number `24`, it is represented as `0b00011000` (or `0B00011000`) in binary, `0o30` (or `0O30`) in octal, and `0x18` (or `0X18`) in hexadecimal. Numbers are decimal by default. For example, `100` can generally be understood as `100` in decimal.

In various base representations, underscores `_` can be used as separators to facilitate identification of the number of digits, such as `0b0001_1000`.

For integer type literals, if their value exceeds the representation range of the integer type required by the context, the compiler will report an error.

```typescript
let x: Int8 = 128; // Error, 128 out of the range of Int8
let y: UInt8 = 256; // Error, 256 out of the range of UInt8
let z: Int32 = 0x8000_0000; // Error, 0x8000_0000 out of the range of Int32
```

When using integer type literals, you can explicitly specify the type of integer literals by adding **suffixes**. The correspondence between suffixes and types is:

| Suffix | Type  | Suffix | Type   |
| :----- | :---- | :----- | :----- |
| i8     | Int8  | u8     | UInt8  |
| i16    | Int16 | u16    | UInt16 |
| i32    | Int32 | u32    | UInt32 |
| i64    | Int64 | u64    | UInt64 |

Integer literals with suffixes can be used in the following ways:

```typescript
var x = 100i8  // x is 100 with type Int8
var y = 0x10u64 // y is 16 with type UInt64
var z = 0o432i32  // z is 282 with type Int32
```

## Character Byte Literals

The Cangjie programming language supports character byte literals to facilitate using ASCII codes to represent `UInt8` type values. Character byte literals consist of the character b, a pair of single quotes marking the beginning and end, and an `ASCII` character, for example:

```typescript
var a = b'x' // a is 120 with type UInt8
var b = b'\n' // b is 10 with type UInt8
var c = b'\u{78}' // c is 120 with type UInt8
```

`b'x'` represents a literal value of type UInt8 with size 120. You can also use the escape form `b'\u{78}'` to represent a literal value of type `UInt8` with hexadecimal size 0x78 or decimal size 120. Note that there can be at most two hexadecimal digits inside `\u`, and the value must be less than 256 (decimal).

### ASCII Table

| Decimal | Hexadecimal | Character |
| ------- | ----------- | --------- |
| 48      | 30          | 0         |
| 49      | 31          | 1         |
| 50      | 32          | 2         |
| 51      | 33          | 3         |
| 52      | 34          | 4         |
| 53      | 35          | 5         |
| 54      | 36          | 6         |
| 55      | 37          | 7         |
| 56      | 38          | 8         |
| 57      | 39          | 9         |
| 65      | 41          | A         |
| 66      | 42          | B         |
| 67      | 43          | C         |
| 68      | 44          | D         |
| 69      | 45          | E         |
| 70      | 46          | F         |
| 71      | 47          | G         |
| 72      | 48          | H         |
| 73      | 49          | I         |
| 74      | 4A          | J         |
| 75      | 4B          | K         |
| 76      | 4C          | L         |
| 77      | 4D          | M         |
| 78      | 4E          | N         |
| 79      | 4F          | O         |
| 80      | 50          | P         |
| 81      | 51          | Q         |
| 82      | 52          | R         |
| 83      | 53          | S         |
| 84      | 54          | T         |
| 85      | 55          | U         |
| 86      | 56          | V         |
| 87      | 57          | W         |
| 88      | 58          | X         |
| 89      | 59          | Y         |
| 90      | 5A          | Z         |
| 97      | 61          | a         |
| 98      | 62          | b         |
| 99      | 63          | c         |
| 100     | 64          | d         |
| 101     | 65          | e         |
| 102     | 66          | f         |
| 103     | 67          | g         |
| 104     | 68          | h         |
| 105     | 69          | i         |
| 106     | 6A          | j         |
| 107     | 6B          | k         |
| 108     | 6C          | l         |
| 109     | 6D          | m         |
| 110     | 6E          | n         |
| 111     | 6F          | o         |
| 112     | 70          | p         |
| 113     | 71          | q         |
| 114     | 72          | r         |
| 115     | 73          | s         |
| 116     | 74          | t         |
| 117     | 75          | u         |
| 118     | 76          | v         |
| 119     | 77          | w         |
| 120     | 78          | x         |
| 121     | 79          | y         |
| 122     | 7A          | z         |
| 33      | 21          | !         |
| 34      | 22          | "         |
| 35      | 23          | #         |
| 36      | 24          | $         |
| 37      | 25          | %         |
| 38      | 26          | &         |
| 39      | 27          | '         |
| 40      | 28          | (         |
| 41      | 29          | )         |
| 42      | 2A          | \*        |
| 43      | 2B          | +         |
| 44      | 2C          | ,         |
| 45      | 2D          | -         |
| 46      | 2E          | .         |
| 47      | 2F          | /         |
| 58      | 3A          | :         |
| 59      | 3B          | ;         |
| 60      | 3C          | <         |
| 61      | 3D          | =         |
| 62      | 3E          | >         |
| 63      | 3F          | ?         |
| 64      | 40          | @         |
| 91      | 5B          | [         |
| 92      | 5C          | \|        |
| 93      | 5D          | ]         |
| 94      | 5E          | ^         |
| 95      | 5F          | \_        |
| 96      | 60          | `         |
| 123     | 7B          | {         |
| 124     | 7C          | \|        |
| 125     | 7D          | }         |
| 126     | 7E          | ~         |

</rewritten_file>
