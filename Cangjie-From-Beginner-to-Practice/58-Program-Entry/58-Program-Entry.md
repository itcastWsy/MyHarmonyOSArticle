# 58-Program Entry Point

The Cangjie program entry point is `main`, and there can be at most one `main` at the top level of packages in the source file root directory.

If the module uses a compilation method that generates executable files, the compiler only looks for `main` at the top level in the source file root directory. If not found, the compiler will report an error; if `main` is found, the compiler will further check its parameter and return value types. Note that `main` cannot be modified by access modifiers, and when a package is imported, the `main` defined in the package will not be imported.

As a program entry point, `main` can have no parameters or have a parameter type of `Array<String>`, with a return value type of `Unit` or integer type.

`main` without parameters:

```typescript
// main.cj
main(): Int64 { // Ok.
    return 0
}
```

`main` with parameter type `Array<String>`:

```typescript
// main.cj
main(args: Array<String>): Unit { // Ok.
    for (arg in args) {
        println(arg)
    }
}
```

After compiling with `cjc main.cj`, execute through command line: `./main Hello, World`, and you will get the following output:

```text
Hello,
World
```

Here are some error examples:

```typescript
// main.cj
main(): String { // Error, return type of 'main' is not 'Integer' or 'Unit'.
    return ""
}
// main.cj
main(args: Array<Int8>): Int64 { // Error, 'main' cannot be defined with parameter whose type is not Array<String>.
    return 0
}
// main.cj
// Error, multiple 'main's are found in source files.
main(args: Array<String>): Int32 {
    return 0
}

main(): Int8 {
    return 0
}
```