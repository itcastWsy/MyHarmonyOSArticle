# 56-Visibility of Top-level Declarations in Packages

In Cangjie, access modifiers can be used to control the visibility of top-level declarations such as types, variables, and functions. Cangjie has 4 access modifiers: `private`, `internal`, `protected`, `public`. The semantics of different access modifiers when modifying top-level elements are as follows:

- `private` means visible only within the current file. Different files cannot access such members.
- `internal` means visible only within the current package and sub-packages (including sub-packages of sub-packages). Members of this type can be accessed without import within the same package, and can be accessed through import within sub-packages of the current package (including sub-packages of sub-packages).
- `protected` means visible only within the current module. Files in the same package can access such members without import, other packages in different packages but within the same module can access these members through import, and packages in different modules cannot access these members.
- `public` means visible both inside and outside the module. Files in the same package can access such members without import, and other packages can access these members through import.

| Modifier    | File | Package & Sub-packages | Module | All Packages |
| ----------- | ---- | ---------------------- | ------ | ------------ |
| `private`   | Y    | N                      | N      | N            |
| `internal`  | Y    | Y                      | N      | N            |
| `protected` | Y    | Y                      | Y      | N            |
| `public`    | Y    | Y                      | Y      | Y            |

The access modifiers supported by different top-level declarations and their default modifiers (default modifiers refer to the modifier semantics when omitted, and these default modifiers can also be explicitly written) are specified as follows:

- `package` supports using `internal`, `protected`, `public`, with the default modifier being `public`.
- `import` supports using all access modifiers, with the default modifier being `private`.
- Other top-level declarations support using all access modifiers, with the default modifier being `internal`.

```typescript
package a

private func f1() { 1 }   // f1 is visible only within the current file
func f2() { 2 }           // f2 is visible only within the current package and sub-packages
protected func f3() { 3 } // f3 is visible only within the current module
public func f4() { 4 }    // f4 is visible both inside and outside the current module
```

Cangjie's access level ordering is `public > protected > internal > private`. The access modifier of a declaration must not be higher than the level of the access modifiers of the types used in that declaration. Refer to the following examples:

- Parameters and return values in function declarations

  ```typescript
  // a.cj
  package a
  class C {}
  public func f1(a1: C) // Error, public declaration f1 cannot use internal type C.
  {
      return 0
  }
  public func f2(a1: Int8): C // Error, public declaration f2 cannot use internal type C.
  {
      return C()
  }
  public func f3 (a1: Int8) // Error, public declaration f3 cannot use internal type C.
  {
      return C()
  }
  ```

- Variable declarations

  ```typescript
  // a.cj
  package a
  class C {}
  public let v1: C = C() // Error, public declaration v1 cannot use internal type C.
  public let v2 = C() // Error, public declaration v2 cannot use internal type C.
  ```

- Inherited classes in class declarations

  ```typescript
  // a.cj
  package a
  open class C1 {}
  public class C2 <: C1 {} // Error, public declaration C2 cannot use internal type C1.
  ```

- Interfaces implemented by types

  ```typescript
  // a.cj
  package a
  interface I {}
  public enum E <: I { A } // Error, public declaration uses internal types.
  ```

- Type arguments of generic types

  ```typescript
  // a.cj
  package a
  public class C1<T> {}
  class C2 {}
  public let v1 = C1<C2>() // Error, public declaration v1 cannot use internal type C2.
  ```

- Type upper bounds in `where` constraints

  ```typescript
  // a.cj
  package a
  interface I {}
  public class B<T> where T <: I {}  // Error, public declaration B cannot use internal type I.
  ```

It's worth noting that:

- Declarations modified by `public` can use any types visible in the current package in their initialization expressions or function bodies, including types modified by `public` and types not modified by `public`.

  ```typescript
  // a.cj
  package a
  class C1 {}
  func f1(a1: C1)
  {
    return 0
  }
  public func f2(a1: Int8) // Ok.
  {
    var v1 = C1()
    return 0
  }
  public let v1 = f1(C1()) // Ok.
  public class C2 // Ok.
  {
    var v2 = C1()
  }
  ```

- Top-level declarations modified by `public` can use anonymous functions, or any top-level functions, including types modified by `public` and top-level functions not modified by `public`.

  ```typescript
  public var t1: () -> Unit = { => } // Ok.
  func f1(): Unit {}
  public let t2 = f1 // Ok.

  public func f2() // Ok.
  {
    return f1
  }
  ```

- Built-in types such as `Rune`, `Int64`, etc. are also `public` by default.

  ```typescript
  var num = 5;
  var t3 = num; // Ok.
  ```