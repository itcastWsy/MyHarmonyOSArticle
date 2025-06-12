# 38-Using Patterns in Other Places

Besides being used in `match` expressions, patterns can also be used in variable definitions (the left side of the equals sign is a pattern) and for-in expressions (between the `for` keyword and `in` keyword is a pattern).

However, not all patterns can be used in variable definitions and `for in` expressions. Only `irrefutable` patterns can be used in these two places, so only wildcard patterns, binding patterns, `irrefutable` tuple patterns, and `irrefutable` enum patterns are allowed.

1. Examples of using wildcard patterns in variable definitions and `for in` expressions are as follows:

   ```javascript
   main() {
       let _ = 100
       for (_ in 1..5) {
           println("0")
       }
   }
   ```

   In the above example, a wildcard pattern is used in the variable definition, indicating that a variable without a name is defined (and therefore cannot be accessed afterwards). A wildcard pattern is used in the `for in` expression, indicating that elements in `1..5` will not be bound to any variable (so the loop body cannot access the element values in `1..5`). Compiling and executing the above code, the output result is:

   ```javascript
   0
   0
   0
   0
   ```

2. Examples of using binding patterns in variable definitions and `for in` expressions are as follows:

   ```javascript
   main() {
       let x = 100
       println("x = ${x}")
       for (i in 1..5) {
           println(i)
       }
   }
   ```

   In the above example, `x` in the variable definition and `i` in the `for in` expression are both binding patterns. Compiling and executing the above code, the output result is:

   ```javascript
   x = 100
   1
   2
   3
   4
   ```

3. Examples of using `irrefutable` tuple patterns in variable definitions and `for in` expressions are as follows:

   ```javascript
   main() {
       let (x, y) = (100, 200)
       println("x = ${x}")
       println("y = ${y}")
       for ((i, j) in [(1, 2), (3, 4), (5, 6)]) {
           println("Sum = ${i + j}")
       }
   }
   ```

   In the above example, a tuple pattern is used in the variable definition, indicating that `(100, 200)` is destructured and bound to `x` and `y` respectively, which is equivalent to defining two variables `x` and `y`. A tuple pattern is used in the `for in` expression, indicating that tuple-type elements in `[(1, 2), (3, 4), (5, 6)]` are taken out sequentially, then destructured and bound to `i` and `j` respectively, and the loop body outputs the value of `i + j`. Compiling and executing the above code, the output result is:

   ```javascript
   x = 100
   y = 200
   Sum = 3
   Sum = 7
   Sum = 11
   ```

4. Examples of using `irrefutable` enum patterns in variable definitions and `for in` expressions are as follows:

   ```javascript
   enum RedColor {
       Red(Int64)
   }
   main() {
       let Red(red) = Red(0)
       println("red = ${red}")
       for (Red(r) in [Red(10), Red(20), Red(30)]) {
           println("r = ${r}")
       }
   }
   ```

   In the above example, an enum pattern is used in the variable definition, indicating that `Red(0)` is destructured and the constructor's parameter value (i.e., `0`) is bound to `red`. An enum pattern is used in the `for in` expression, indicating that elements in `[Red(10), Red(20), Red(30)]` are taken out sequentially, then destructured and the constructor's parameter values are bound to `r`, and the loop body outputs the value of `r`. Compiling and executing the above code, the output result is:

   ```javascript
   red = 0
   r = 10
   r = 20
   r = 30
   ```