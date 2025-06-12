# 55-Package Declaration

## Package Introduction

When project scale continues to expand, managing source code in a single huge file is like finding a small item in a chaotic giant warehouse - extremely difficult. At this time, grouping source code according to functionality and managing code with different functions separately is a good solution. Each group of independently managed code will eventually generate an output file. In actual use, importing the corresponding output file allows you to call the corresponding functionality, or achieve more complex features through interaction and combination of different functions, greatly improving project management efficiency.

In the Cangjie programming language, packages are called the **minimum unit of compilation**. Each package can independently output AST files, static library files, dynamic library files, and other results. Each package has its own namespace, and within the same package, except for function overloading situations, top-level definitions or declarations with the same name are not allowed. A package can also contain multiple source files. A module is a collection of several packages, and it is the **minimum unit for third-party developers to publish**.

The program entry point of a module is limited to its root directory, and there can be at most one `main` at the top level that serves as the program entry point. This `main` either has no parameters, or has a parameter type of `Array<String>`, with a return type of integer type or `Unit` type.

## Package Declaration

In the Cangjie programming language, package declarations start with the keyword `package`, followed by the package names of all packages on the path from the root package to the current package separated by `.`. Package names must be valid ordinary identifiers (not including raw identifiers). For example:

```typescript
package pkg1      // root package pkg1
package pkg1.sub1 // sub-package sub1 of root package pkg1
```

Package declarations must be on the first non-empty, non-comment line of the source file, and package declarations in different source files of the same package must be consistent.

```typescript
// file 1
// Comments are accepted
package test
// declarations...

// file 2
let a = 1 // Error, package declaration must appear first in a file
package test
// declarations...
```

Cangjie package names need to reflect the path of the current source file relative to the project source root directory `src`, replacing the path separators with dots. For example, if the package's source code is located under `src/directory_0/directory_1` and the root package name is `pkg`, then the package declaration in its source code should be `package pkg.directory_0.directory_1`.

Note that:

- The folder name where the package is located must be consistent with the package name.
- The source root directory is named `src` by default.
- Packages under the source root directory can have no package declaration, in which case the compiler will default to specifying the package name `default`.

Assume the source code directory structure is as follows:

```text
// The directory structure is as follows:
src
`-- directory_0
    |-- directory_1
    |    |-- a.cj
    |    `-- b.cj
    `-- c.cj
`-- main.cj
```

Then the package declarations in `a.cj`, `b.cj`, `c.cj`, `main.cj` can be:

```typescript
// a.cj
// in file a.cj, the declared package name must correspond to relative path directory_0/directory_1.

package default.directory_0.directory_1
// b.cj
// in file b.cj, the declared package name must correspond to relative path directory_0/directory_1.

package default.directory_0.directory_1
// c.cj
// in file c.cj, the declared package name must correspond to relative path directory_0.

package default.directory_0
// main.cj
// file main.cj is in the module root directory and may omit package declaration.

main() {
    return 0
}
```

Additionally, package declarations cannot cause naming conflicts: sub-packages cannot have the same name as top-level declarations of the current package.

Here are some error examples:

```typescript
// a.cj
package a
public class B { // Error, 'B' is conflicted with sub-package 'a.B'
    public static func f() {}
}

// b.cj
package a.B
public func f {}

// main.cj
import a.B // ambiguous use of 'a.B'

main() {
    a.B.f()
    return 0
}
```