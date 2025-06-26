# 13-TypeScript to ArkTS Migration Rules (Part 3)

### Type Conversion Only Supports "as T" Syntax

**Rule:** arkts-as-casts

**Level:** Error

In ArkTS, the `as` keyword is the only syntax for type conversion. Incorrect type conversion will cause compilation errors or throw ClassCastException at runtime. ArkTS does not support using `<type>` syntax for type conversion.

When you need to convert primitive types (such as number or boolean) to reference types, please use the `new` expression.

**TypeScript**

```typescript
class Shape {}
class Circle extends Shape {
  x: number = 5;
}
class Square extends Shape {
  y: string = "a";
}
function createShape(): Shape {
  return new Circle();
}
let c1 = <Circle>createShape();
let c2 = createShape() as Circle;
// If the conversion is wrong, it will not cause compilation or runtime errorlet c3 = createShape() as Square;console.log(c3.y); // undefined
// In TS, since the `as` keyword does not take effect at runtime, the left operand of `instanceof` will not be boxed into a reference type at runtimelet e1 = (5.0 as Number) instanceof Number; // false
// Create a Number object to get the expected result:let e2 = (new Number(5.0)) instanceof Number; // true
```

**ArkTS**

```typescript
class Shape {}
class Circle extends Shape {
  x: number = 5;
}
class Square extends Shape {
  y: string = "a";
}
function createShape(): Shape {
  return new Circle();
}
let c2 = createShape() as Circle;
// Throws ClassCastException at runtime:let c3 = createShape() as Square;
// Create a Number object to get the expected result:let e2 = (new Number(5.0)) instanceof Number; // true
```

### JSX Expressions Are Not Supported

**Rule:** arkts-no-jsx

**Level:** Error

JSX is not supported.

### Unary Operators +, -, and ~ Only Apply to Numeric Types

**Rule:** arkts-no-polymorphic-unops

**Level:** Error

ArkTS only allows unary operators to be used with numeric types, otherwise compilation errors will occur. Unlike TypeScript, ArkTS does not support implicit conversion of strings to numbers and must perform explicit conversion.

**TypeScript**

```typescript
let a = +5; // 5 (number type)let b = +'5';    // 5 (number type)let c = -5;    // -5 (number type)let d = -'5';    // -5 (number type)let e = ~5;    // -6 (number type)let f = ~'5';    // -6 (number type)let g = +'string'; // NaN (number type)
function returnTen(): string {
  return "-10";
}
function returnString(): string {
  return "string";
}
let x = +returnTen(); // -10 (number type)let y = +returnString(); // NaN
```

**ArkTS**

```typescript
let a = +5; // 5 (number type)let b = +'5';    // Compilation errorlet c = -5;    // -5 (number type)let d = -'5';    // Compilation errorlet e = ~5;    // -6 (number type)let f = ~'5';    // Compilation errorlet g = +'string'; // Compilation error
function returnTen(): string {
  return "-10";
}
function returnString(): string {
  return "string";
}
let x = +returnTen(); // Compilation errorlet y = +returnString(); // Compilation error
```

### The delete Operator Is Not Supported

**Rule:** arkts-no-delete

**Level:** Error

In ArkTS, object layout is determined at compile time and cannot be changed at runtime. Therefore, deleting properties is meaningless.

**TypeScript**

```typescript
class Point {  x?: number = 0.0  y?: number = 0.0}
let p = new Point();delete p.y;
```

**ArkTS**

```typescript
// You can declare a nullable type and use null as the default valueclass Point {  x: number | null = 0  y: number | null = 0}
let p = new Point();
p.y = null;
```

### The typeof Operator Is Only Allowed in Expressions

**Rule:** arkts-no-type-query

**Level:** Error

ArkTS only supports using the `typeof` operator in expressions and does not allow using `typeof` as a type.

**TypeScript**

```typescript
let n1 = 42;
let s1 = "foo";
console.log(typeof n1); // 'number'console.log(typeof s1); // 'string'let n2: typeof n1let s2: typeof s1
```

**ArkTS**

```typescript
let n1 = 42;
let s1 = "foo";
console.log(typeof n1); // 'number'console.log(typeof s1); // 'string'let n2: numberlet s2: string
```

### Partial Support for the instanceof Operator

**Rule:** arkts-instanceof-ref-types

**Level:** Error

In TypeScript, the type of the left operand of the `instanceof` operator must be any type, object type, or it is a type parameter, otherwise the result is false. In ArkTS, the type of the left operand of the `instanceof` operator must be a reference type (for example, object, array, or function), otherwise a compilation error will occur. In addition, in ArkTS, the left operand of the `instanceof` operator cannot be a type, it must be an instance of an object.

### The in Operator Is Not Supported

**Rule:** arkts-no-in

**Level:** Error

Since object layout is known at compile time and cannot be modified at runtime in ArkTS, the `in` operator is not supported. If you still need to check whether certain class members exist, use `instanceof` instead.

**TypeScript**

```typescript
class Person {
  name: string = "";
}
let p = new Person();
let b = "name" in p; // true
```

**ArkTS**

```typescript
class Person {
  name: string = "";
}
let p = new Person();
let b = p instanceof Person; // true, and property name definitely exists
```

### Destructuring Assignment Is Not Supported

**Rule:** arkts-no-destruct-assignment

**Level:** Error

ArkTS does not support destructuring assignment. You can use alternative methods, such as using temporary variables.

**TypeScript**

```typescript
let [one, two] = [1, 2]; // Semicolon needed here[one, two] = [two, one];
let head, tail[head, ...tail] = [1, 2, 3, 4];
```

**ArkTS**

```typescript
let arr: number[] = [1, 2];
let one = arr[0];
let two = arr[1];
let tmp = one;
one = two;
two = tmp;
let data: Number[] = [1, 2, 3, 4];
let head = data[0];
let tail: Number[] = [];
for (let i = 1; i < data.length; ++i) {
  tail.push(data[i]);
}
```

### The Comma Operator , Is Only Used in for Loop Statements

**Rule:** arkts-no-comma-outside-loops

**Level:** Error

To facilitate understanding of execution order, in ArkTS, the comma operator is only applicable in for loop statements. Note that this is different from comma separators when declaring variables and passing function parameters.

**TypeScript**

```typescript
for (let i = 0, j = 0; i < 10; ++i, j += 2) {  // ...}
let x = 0;x = (++x, x++); // 1
```

**ArkTS**

```typescript
for (let i = 0, j = 0; i < 10; ++i, j += 2) {  // ...}
// Express execution order through statements, not comma operatorlet x = 0;++x;x = x++;
```

### Destructuring Variable Declarations Are Not Supported

**Rule:** arkts-no-destruct-decls

**Level:** Error

ArkTS does not support destructuring variable declarations. It is a dynamic feature that depends on structural compatibility and the names in destructuring declarations must be consistent with the property names in the destructured object.

**TypeScript**

```typescript
class Point {  x: number = 0.0  y: number = 0.0}
function returnZeroPoint(): Point {  return new Point();}
let {x, y} = returnZeroPoint();
```

**ArkTS**

```typescript
class Point {  x: number = 0.0  y: number = 0.0}
function returnZeroPoint(): Point {  return new Point();}
// Create a local variable to handle each fieldlet zp = returnZeroPoint();let x = zp.x;let y = zp.y;
```

### Type Annotations in catch Statements Are Not Supported

**Rule:** arkts-no-types-in-catch

**Level:** Error

In TypeScript's catch statements, only any or unknown types can be annotated. Since ArkTS does not support these types, type annotations should be omitted.

**TypeScript**

```typescript
try {  // ...} catch (a: unknown) {  // Handle exception}
```

**ArkTS**

```typescript
try {  // ...} catch (a) {  // Handle exception}
```

### for .. in Is Not Supported

**Rule:** arkts-no-for-in

**Level:** Error

Since object layout is determined at compile time and cannot be changed at runtime in ArkTS, using for .. in to iterate over object properties is not supported. For arrays, you can use regular for loops.

**TypeScript**

```typescript
let a: string[] = ["1.0", "2.0", "3.0"];
for (let i in a) {
  console.log(a[i]);
}
```

**ArkTS**

```typescript
let a: string[] = ["1.0", "2.0", "3.0"];
for (let i = 0; i < a.length; ++i) {
  console.log(a[i]);
}
```

### Mapped Types Are Not Supported

**Rule:** arkts-no-mapped-types

**Level:** Error

ArkTS does not support mapped types. Use other syntax to express the same semantics.

**TypeScript**

```typescript
type OptionsFlags<Type> = { [Property in keyof Type]: boolean };
```

**ArkTS**

```typescript
class C {  n: number = 0  s: string = ''}
class CFlags {  n: boolean = false  s: boolean = false}
```

### with Statements Are Not Supported

**Rule:** arkts-no-with

**Level:** Error

ArkTS does not support with statements. Use other syntax to express the same semantics.

**TypeScript**

```typescript
with (Math) { // Compilation error, but JavaScript code can still be generated  let r: number = 42;  let area: number = PI * r * r;}
```

**ArkTS**

```typescript
let r: number = 42;
let area: number = Math.PI * r * r;
```

### Limit Expression Types in throw Statements

**Rule:** arkts-limited-throw

**Level:** Error

ArkTS only supports throwing instances of the Error class or its derived classes. Throwing data of other types (such as number or string) is prohibited.

**TypeScript**

```typescript
throw 4;
throw "";
throw new Error();
```

**ArkTS**

```typescript
throw new Error();
```

### Limit Omission of Function Return Type Annotations

**Rule:** arkts-no-implicit-return-types

**Level:** Error

ArkTS supports inference of function return types in some scenarios. When the expression in a return statement is a call to a function or method, and the return type of that function or method is not explicitly annotated, a compilation error will occur. In this case, please annotate the function return type.

**TypeScript**

```typescript
// Only produces compilation error when noImplicitAny option is enabledfunction f(x: number) {  if (x <= 0) {    return x;  }  return g(x);}
// Only produces compilation error when noImplicitAny option is enabledfunction g(x: number) {  return f(x - 1);}
function doOperation(x: number, y: number) {
  return x + y;
}
f(10);
doOperation(2, 3);
```

**ArkTS**

```typescript
// Return type annotation required:function f(x: number): number {  if (x <= 0) {    return x;  }  return g(x);}
// Return type can be omitted, return type can be inferred from f's type annotationfunction g(x: number): number {  return f(x - 1);}
// Return type can be omittedfunction doOperation(x: number, y: number) {  return x + y;}
f(10);
doOperation(2, 3);
```

### Function Declarations with Parameter Destructuring Are Not Supported

**Rule:** arkts-no-destruct-params

**Level:** Error

ArkTS requires that actual parameters must be passed directly to functions and must be specified to formal parameters.

**TypeScript**

```typescript
function drawText({ text = "", location: [x, y] = [0, 0], bold = false }) {
  text;
  x;
  y;
  bold;
}
drawText({ text: "Hello, world!", location: [100, 50], bold: true });
```

**ArkTS**

```typescript
function drawText(text: String, location: number[], bold: boolean) {
  let x = location[0];
  let y = location[1];
  text;
  x;
  y;
  bold;
}
function main() {
  drawText("Hello, world!", [100, 50], true);
}
```

### Declaring Functions Inside Functions Is Not Supported

**Rule:** arkts-no-nested-funcs

**Level:** Error

ArkTS does not support declaring functions inside functions. Use lambda functions instead.

**TypeScript**

```typescript
function addNum(a: number, b: number): void {
  // Declare function inside function  function logToConsole(message: string): void {    console.log(message);  }
  let result = a + b;
  // Call function  logToConsole('result is ' + result);}
```

**ArkTS**

```typescript
function addNum(a: number, b: number): void {
  // Use lambda function instead of declaring function  let logToConsole: (message: string) => void = (message: string): void => {    console.log(message);  }
  let result = a + b;
  logToConsole("result is " + result);
}
```

### Using this in Functions and Static Methods of Classes Is Not Supported

**Rule:** arkts-no-standalone-this

**Level:** Error

ArkTS does not support using `this` in functions and static methods of classes. `this` can only be used in instance methods of classes.

**TypeScript**

```typescript
function foo(i: string) {  this.count = i; // Only produces compilation error when noImplicitThis option is enabled}
class A {  count: string = 'a'  m = foo}
let a = new A();console.log(a.count); // Prints aa.m('b');console.log(a.count); // Prints b
```

**ArkTS**

```typescript
class A {  count: string = 'a'  m(i: string): void {    this.count = i;  }}
function main(): void {  let a = new A();  console.log(a.count);  // Prints a  a.m('b');  console.log(a.count);  // Prints b}
```
