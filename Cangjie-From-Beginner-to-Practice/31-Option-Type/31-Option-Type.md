# 31-Option Type

## Introduction

`Option` can be understood as a wrapper for commonly used data types, with two possible results:

1. **Has value** `Some`
2. **No value** `None`

The `Option` type is defined using `enum`, containing two constructors: `Some` and `None`. Among them, `Some` carries a parameter indicating there is a value, while `None` has no parameters, indicating no value. When you need to represent that a certain type might have a value or might not have a value, you can choose to use the `Option` type.

## Example

The `Option` type is defined as a generic `enum` type:

```javascript
enum Option<T> {
    | Some(T)
    | None
}
```

The following example shows how to define variables of `Option` type:

```javascript
let a: Option<Int64> = Some(100);
let b: ?Int64 = Some(100);
let c: Option<String> = Some("Hello");
let d: ?String = None;
```

The compiler will use the `Some` constructor of `Option<T>` type to wrap a value of type `T` into a value of type `Option<T>` (note: this is not type conversion). For example, the following definitions are legal (equivalent to the definitions of variables `a`, `b`, and `c` in the above example):

```javascript
let a: Option<Int64> = 100;
let b: ?Int64 = 100;
let c: Option<String> = "100";
```

When the context has no explicit type requirements, you cannot use `None` directly to construct the desired type. In this case, you should use syntax like `None<T>` to construct data of type `Option<T>`, for example:

```javascript
let a = None<Int64> // a: Option<Int64>
let b = None<Bool> // b: Option<Bool>
```
