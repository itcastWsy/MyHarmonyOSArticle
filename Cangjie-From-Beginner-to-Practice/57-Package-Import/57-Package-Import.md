# 57-Package Import

## Using import statements to import declarations or definitions from other packages

In the Cangjie programming language, you can import a top-level declaration or definition from other packages using the syntax `import fullPackageName.itemName`, where `fullPackageName` is the complete package path name and `itemName` is the name of the declaration. Import statements must be positioned after package declarations and before other declarations or definitions in the source file. For example:

```typescript
package a
import std.math.*
import package1.foo
import {package1.foo, package2.bar}
```

If multiple `itemName`s to be imported belong to the same `fullPackageName`, you can use the syntax `import fullPackageName.{itemName[, itemName]*}`, for example:

```typescript
import package1.{foo, bar, fuzz}
```

This is equivalent to:

```typescript
import package1.foo
import package1.bar
import package1.fuzz
```

Besides importing a specific top-level declaration or definition through the `import fullPackagename.itemName` syntax, you can also use the `import packageName.*` syntax to import all visible top-level declarations or definitions from the `packageName` package. For example:

```typescript
import package1.*
import {package1.*, package2.*}
```

Note that:

- The scope level of imported members is lower than that of members declared in the current package.
- When the module name or package name of an exported package is tampered with, making it inconsistent with the module name or package name specified during export, an error will be reported during import.
- Only top-level declarations or definitions visible to the current file are allowed to be imported; importing invisible declarations or definitions will cause an error at the import location.
- It is forbidden to import declarations or definitions from the package where the current source file is located through `import`.
- Circular dependency imports between packages are forbidden; if there are circular dependencies between packages, the compiler will report an error.

Examples are as follows:

```typescript
// pkga/a.cj
package pkga    // Error, packages pkga pkgb are in circular dependencies.
import pkgb.*

class C {}
public struct R {}

// pkgb/b.cj
package pkgb

import pkga.*

// pkgc/c1.cj
package pkgc

import pkga.C // Error, 'C' is not accessible in package 'pkga'.
import pkga.R // OK, R is an external top-level declaration of package pkga.
import pkgc.f1 // Error, package 'pkgc' should not import itself.

public func f1() {}

// pkgc/c2.cj
package pkgc

func f2() {
    /* OK, the imported declaration is visible to all source files of the same package
     * and accessing import declaration by its name is supported.
     */
    R()

    // OK, accessing imported declaration by fully qualified name is supported.
    pkga.R()

    // OK, the declaration of current package can be accessed directly.
    f1()

    // OK, accessing declaration of current package by fully qualified name is supported.
    pkgc.f1()
}
```

In the Cangjie programming language, if imported declarations or definitions have the same name as top-level declarations or definitions in the current package and do not constitute function overloading, the imported declarations and definitions will be shadowed; if imported declarations or definitions have the same name as top-level declarations or definitions in the current package and constitute function overloading, function resolution will be performed according to function overloading rules during function calls.

```typescript
// pkga/a.cj
package pkga

public struct R {}            // R1
public func f(a: Int32) {}    // f1
public func f(a: Bool) {} // f2

// pkgb/b.cj
package pkgb
import pkga.*

func f(a: Int32) {}         // f3
struct R {}                 // R2

func bar() {
    R()     // OK, R2 shadows R1.
    f(1)    // OK, invoke f3 in current package.
    f(true) // OK, invoke f2 in the imported package
}
```

## Implicit import of core package

Types such as `String`, `Range`, etc. can be used directly, not because these types are built-in types, but because the compiler automatically imports all `public` modified declarations from the `core` package implicitly for the source code.

## Using `import as` to rename imported names

Namespaces of different packages are separated, so there may be top-level declarations with the same name in different packages. When importing top-level declarations with the same name from different packages, you can use `import packageName.name as newName` to rename and avoid conflicts. Even when there are no name conflicts, you can still use `import as` to rename imported content. `import as` has the following rules:

- After renaming imported declarations using `import as`, the current package can only use the new renamed name, and the original name cannot be used.

- If the renamed name conflicts with other names in the top-level scope of the current package, and the declarations corresponding to these names are all function types, they participate in function overloading; otherwise, a redefinition error is reported.

- The form `import pkg as newPkgName` is supported to rename package names to resolve naming conflicts of packages with the same name in different modules.

  ```typescript
  // a.cj
  package p1
  public func f1() {}
  
  // d.cj
  package p2
  public func f3() {}
  
  // b.cj
  package p1
  public func f2() {}
  
  // c.cj
  package pkgc
  public func f1() {}
  
  // main.cj
  import p1 as A
  import p1 as B
  import p2.f3 as f  // OK
  import pkgc.f1 as a
  import pkgc.f1 as b // OK
  
  func f(a: Int32) {}
  
  main() {
      A.f1()  // OK, package name conflict is resolved by renaming package name.
      B.f2()  // OK, package name conflict is resolved by renaming package name.
      p1.f1() // Error, the original package name cannot be used.
      a()     // Ok.
      b()     // Ok.
      pkgc.f1()    // Error, the original name cannot be used.
  }
  ```

- If conflicting imported names are not renamed, no error is reported at the `import` statement; at the usage location, an error will be reported because a unique name cannot be imported. This situation can be resolved by defining aliases through `import as` or importing packages as namespaces through `import fullPackageName`.

  ```typescript
  // a.cj
  package p1
  public class C {}
  
  // b.cj
  package p2
  public class C {}
  
  // main1.cj
  package pkga
  import p1.C
  import p2.C
  
  main() {
      let _ = C() // Error
  }
  
  // main2.cj
  package pkgb
  import p1.C as C1
  import p2.C as C2
  
  main() {
      let _ = C1() // Ok
      let _ = C2() // Ok
  }
  
  // main3.cj
  package pkgc
  import p1
  import p2
  
  main() {
      let _ = p1.C() // Ok
      let _ = p2.C() // Ok
  }
  ```

## Re-exporting an imported name

In the development process of large projects with many features, this scenario is very common: package `p2` extensively uses declarations imported from package `p1`. When package `p3` imports package `p2` and uses its functionality, declarations from `p1` also need to be visible to package `p3`. If package `p3` is required to import declarations from `p1` used in `p2` by itself, this process would be too cumbersome. Therefore, it is hoped that when `p2` is imported, declarations from `p1` used by `p2` can be imported together.

In the Cangjie programming language, `import` can be modified by access modifiers `private`, `internal`, `protected`, `public`. Among them, `import` modified by `public`, `protected`, or `internal` can re-export imported members (if these imported members are not unavailable in the current package due to name conflicts or being shadowed). Other packages can directly import and use re-exported content from the current package according to visibility, without needing to import this content from the original package.

- `private import` means the imported content is accessible only within the current file. `private` is the default modifier for `import`; `import` without an access modifier is equivalent to `private import`.
- `internal import` means the imported content is accessible within the current package and its sub-packages (including sub-packages of sub-packages). Access from non-current packages requires explicit `import`.
- `protected import` means the imported content is accessible within the current module. Access from non-current packages requires explicit `import`.
- `public import` means the imported content is accessible externally. Access from non-current packages requires explicit `import`.

In the following example, `b` is a sub-package of `a`, and function `f` defined in `b` is re-exported in `a` through `public import`.

```typescript
package a
public import a.b.f

public let x = 0
internal package a.b

public func f() { 0 }
import a.f  // Ok
let _ = f() // Ok
```

Note that packages cannot be re-exported: if what is imported by `import` is a package, then that `import` is not allowed to be modified by `public`, `protected`, or `internal`.

```typescript
public import a.b // Error, cannot re-export package
```