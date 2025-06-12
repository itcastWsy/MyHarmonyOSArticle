# 04-Identifiers and Keywords

In the Cangjie language, the names we use for custom variables, functions, classes, and other elements must use **identifiers**. We can also think of variables, functions, classes, etc., as names we give to some children in the world of Cangjie. Since we're naming things, there are certain ranges and restrictions. For example: we wouldn't name a child **the person who sells 75 yuan eyebrow pencils**. Such a name is **illegal and unreasonable**.

## Identifiers are Classified into Two Types

1. Regular identifiers
2. Raw identifiers

## Regular Identifiers

Regular identifiers have the following two naming rules:

1. Two components **[AB]**: **[A]** part starts with English letters, **[B]** can be English letters, numbers, and underscores
2. Three components **[ABC]**: **[A]** part is one or more underscores, **[B]** part is English letters, **[C]** part can be English letters, numbers, and underscores

### Positive Examples

```c++
package project1 // Mark the current project name

main() {
    let abc = "Xiaowan1"
    let abc123 = "Xiaowan"

    let _abc = "Xiaowan"
    let __ = "Xiaowan" // Two underscores

    let _abc123 = "Xiaowan"
}
```

### Negative Examples

```c++
    let 123="Xiaowan"
    let *123="Xiaowan"
    let )123="Xiaowan"
    let +123="Xiaowan"
    let _="Xiaowan"// 1 underscore - can be declared, but will show error when printing
```

## Raw Identifiers

Raw identifiers are not used much in actual development, mainly used in scenarios where Cangjie keywords are used as identifiers.

Can be understood as allowing you to define some special or unusual names.

**Raw identifiers** are **regular identifiers** or **Cangjie keywords** surrounded by a pair of backticks

Such as:

```
`abc`
`if`
`else`
```

### Positive Examples

```c++
package project1 // Mark the current project name

// Struct if as an unusual name
struct `if` {
    let name = "Xiaowan"
}

main() {
    let a: `if` = `if`()
    println(a.name)

    println()
}
```

## Keywords

In the Cangjie language, it has reserved some names as Cangjie keywords or reserved words, meaning they have special purposes and cannot be used by developers as regular identifiers.

Such as **let**:

```
let let = "Xiaowan" // Here let is incorrect usage because let is a keyword
```

### Keywords Overview

| Keyword      | Keyword    | Keyword   |
| ------------ | ---------- | --------- |
| as           | abstract   | break     |
| Bool         | case       | catch     |
| class        | const      | continue  |
| Rune         | do         | else      |
| enum         | extend     | for       |
| func         | false      | finally   |
| foreign      | Float16    | Float32   |
| Float64      | if         | in        |
| is           | init       | import    |
| interface    | Int8       | Int16     |
| Int32        | Int64      | IntNative |
| let          | mut        | main      |
| macro        | match      | Nothing   |
| open         | operator   | override  |
| prop         | public     | package   |
| private      | protected  | quote     |
| redef        | return     | spawn     |
| super        | static     | struct    |
| synchronized | try        | this      |
| true         | type       | throw     |
| This         | unsafe     | Unit      |
| UInt8        | UInt16     | UInt32    |
| UInt64       | UIntNative | var       |
| VArray       | where      | while     |

## Summary

The purpose of identifiers is to tell us how to name things reasonably. Everyone should treat it as normal naming conventions.
