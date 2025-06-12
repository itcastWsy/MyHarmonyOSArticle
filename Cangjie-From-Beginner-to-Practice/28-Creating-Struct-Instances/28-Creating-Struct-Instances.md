# 28-Creating struct Instances

After defining a `struct` type, you can create `struct` instances by calling the `struct`'s constructor. Outside the `struct` definition, the constructor is called through the `struct` type name. For example, the following example defines a variable `r` of type `Rectangle`.

```javascript
let r = Rectangle(10, 20)
```

After creating a `struct` instance, you can access its (`public` modified) instance member variables and instance member functions through the instance. For example, in the following example, you can access the values of `width` and `height` in `r` through `r.width` and `r.height` respectively, and call the member function `area` of `r` through `r.area()`.

```javascript
let r = Rectangle(10, 20)
let width = r.width   // width = 10
let height = r.height // height = 20
let a = r.area()      // a = 200
```

If you want to modify the value of member variables through a `struct` instance, you need to define the variable of `struct` type as a mutable variable, and the member variable being modified must also be a mutable member variable (defined using `var`). Here's an example:

```javascript
struct Rectangle {
    public var width: Int64
    public var height: Int64

    public init(width: Int64, height: Int64) {
        this.width = width
        this.height = height
    }

    public func area() {
        width * height
    }
}

main() {
    var r = Rectangle(10, 20) // r.width = 10, r.height = 20
    r.width = 8               // r.width = 8
    r.height = 24             // r.height = 24
    let a = r.area()          // a = 192
}
```

During assignment or parameter passing, `struct` instances are copied, generating new instances. **Modifications to one instance do not affect another instance**. Taking assignment as an example, in the following example, after assigning `r1` to `r2`, modifying the values of `width` and `height` of `r1` does not affect the values of `width` and `height` of `r2`.

```javascript
struct Rectangle {
    public var width: Int64
    public var height: Int64

    public init(width: Int64, height: Int64) {
        this.width = width
        this.height = height
    }

    public func area() {
        width * height
    }
}

main() {
    var r1 = Rectangle(10, 20) // r1.width = 10, r1.height = 20
    var r2 = r1                // r2.width = 10, r2.height = 20
    r1.width = 8               // r1.width = 8
    r1.height = 24             // r1.height = 24
    let a1 = r1.area()         // a1 = 192
    let a2 = r2.area()         // a2 = 200
}
```