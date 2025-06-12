# 10-Boolean Type

The Boolean type is represented by `Bool` and is used to represent logical true and false values.

## Boolean Type Literals

The Boolean type has only two literals: `true` and `false`.

The following example demonstrates the use of Boolean literals:

```javascript
let a: Bool = true;
let b: Bool = false;
```

## Operations Supported by Boolean Type

Operators supported by the Boolean type include: logical operators (logical NOT `!`, logical AND `&&`, logical OR `||`), some relational operators (`==` and `!=`), assignment operators, and some compound assignment operators (`&&=` and `||=`).

```javascript
package pro

main() {
    let a = true
    // Logical NOT   represents negation
    let b = !a //  false

    // Logical AND
    // let c = !b && a // true    requires all conditions to be met for the result to be true
    // Logical OR
    let d = a || b // true // if one is true, the result is true

    // ==  equality check
    let e = 100 == 200 // false  because 100 is not equal to 200

    // !=  inequality check
    let f = 100 != 200 // true  because 100 is not equal to 200

    // &&=
    var g = false

    g &&= true // g = false    because when g equals false, the true on the right side of &&= is not executed

    var h = true
    h &&= false // h = false  because when h = true, the false on the right side of &&= is executed

    // ||=
    var i = true
    i ||= false // i = true, because when i equals true, the false on the right side of ||= is not executed

    var j = false
    j ||= true // j = true, because when j equals false, the true on the right side of ||= is executed
}

```

**Note that when there are unused variables, the program will generate warnings during compilation**

```javascript
 let e = 100 == 200 // false  because 100 is not equal to 200
  ^ unused variable
  # note: this warning can be suppressed by setting the compiler option `-Woff unused`
```