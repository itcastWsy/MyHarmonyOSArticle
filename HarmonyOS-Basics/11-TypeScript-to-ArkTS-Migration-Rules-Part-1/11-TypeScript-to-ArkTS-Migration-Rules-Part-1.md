# 11-TypeScript to ArkTS Migration Rules (Part 1)

For students who have learned TypeScript and need to migrate to ArkTS, it's important to note that some syntax is not supported. The main characteristic is the removal of dynamic property capabilities.

### Mandatory Use of Static Types

Static typing is one of the most important features of ArkTS. If a program adopts static typing, meaning all types are known at compile time, developers can easily understand what data structures are used in the code. Additionally, since all types are known before the program actually runs, the compiler can validate code correctness in advance, thereby reducing runtime type checking and helping improve performance.

Based on these considerations, ArkTS prohibits the use of the any type.

**Example**

```typescript
// Not supported: let res: any = some_api_function('hello', 'world');// What is `res`? An error code number? A string? An object?// How should we handle it?// Supported:class CallResult {  public succeeded(): boolean { ... }  public errorMessage(): string { ... }}
let res: CallResult = some_api_function("hello", "world");
if (!res.succeeded()) {
  console.log("Call failed: " + res.errorMessage());
}
```

The any type is not common in TypeScript, with only about 1% of TypeScript codebases using it. Some code checking tools (such as ESLint) also establish rules to prohibit the use of any. Therefore, although prohibiting any will lead to code refactoring, the amount of refactoring is small and helps improve overall performance.

### Prohibit Runtime Object Layout Changes

To achieve optimal performance, ArkTS requires that object layouts cannot be changed during program execution. In other words, ArkTS prohibits the following behaviors:

- Adding new properties or methods to objects.
- Deleting existing properties or methods from objects.
- Assigning values of arbitrary types to object properties.

The TypeScript compiler already prohibits many such operations. However, some operations can still bypass the compiler, for example, using as any to convert object types, or turning off strict type checking configuration when compiling TS code, or ignoring type checking in code through @ts-ignore.

In ArkTS, strict type checking is not a configurable option. ArkTS enforces partial strict type checking and prohibits the use of any type through specifications, and prohibits the use of @ts-ignore in code.

**Example**

```typescript
class Point {  public x: number = 0  public y: number = 0
  constructor(x: number, y: number) {    this.x = x;    this.y = y;  }}
// Cannot delete a property from an object, ensuring all Point objects have property xlet p1 = new Point(1.0, 1.0);delete p1.x;           // Produces compile-time error in both TypeScript and ArkTSdelete (p1 as any).x;  // No error in TypeScript; produces compile-time error in ArkTS
// The Point class does not define a property named z, nor can this property be added at runtimelet p2 = new Point(2.0, 2.0);p2.z = 'Label';           // Produces compile-time error in both TypeScript and ArkTS(p2 as any).z = 'Label';   // No error in TypeScript; produces compile-time error in ArkTS
// The class definition ensures that all Point objects only have properties x and y, and no other properties can be addedlet p3 = new Point(3.0, 3.0);let prop = Symbol();      // No error in TypeScript; produces compile-time error in ArkTS(p3 as any)[prop] = p3.x; // No error in TypeScript; produces compile-time error in ArkTSp3[prop] = p3.x;          // Produces compile-time error in both TypeScript and ArkTS
// The class definition ensures that properties x and y of all Point objects have number type, so values of other types cannot be assigned to themlet p4 = new Point(4.0, 4.0);p4.x = 'Hello!';          // Produces compile-time error in both TypeScript and ArkTS(p4 as any).x = 'Hello!'; // No error in TypeScript; produces compile-time error in ArkTS
// Using Point objects that conform to the class definition:function distance(p1: Point, p2: Point): number {  return Math.sqrt(    (p2.x - p1.x) * (p2.x - p1.x) + (p2.y - p1.y) * (p2.y - p1.y)  );}let p5 = new Point(5.0, 5.0);let p6 = new Point(6.0, 6.0);console.log('Distance between p5 and p6: ' + distance(p5, p6));
```

Modifying object layouts affects code readability and runtime performance. From a developer's perspective, defining a class in one place and then modifying the actual object layout elsewhere can easily cause confusion and introduce errors. Additionally, this requires additional runtime support, increasing execution overhead. This also conflicts with static type constraints: if we've decided to use explicit types, why would we need to add or delete properties?

Currently, only a few projects allow runtime object layout changes, and some commonly used code checking tools have also added corresponding restriction rules. This constraint will only cause a small amount of code refactoring but will improve performance.

### Restrict Operator Semantics

To achieve better performance and encourage developers to write clearer code, ArkTS restricts the semantics of some operators. For detailed semantic restrictions, please refer to [Constraint Specifications](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/typescript-to-arkts-migration-guide-V5#约束说明).

**Example**

```typescript
// Unary operator `+` can only be applied to numeric types:let t = +42;   // Legal operationlet s = +'42'; // Compile-time error
```

Using additional semantic overloading of language operators increases the complexity of the language specification, and developers are forced to remember all possible exceptions and corresponding handling rules. In some cases, this creates unnecessary runtime overhead.

Currently, less than 1% of codebases use this feature. Therefore, although restricting operator semantics requires code refactoring, the amount of refactoring is small and very easy to operate, and through refactoring, code becomes clearer and has higher performance.

### No Support for Structural Typing

Suppose two unrelated classes T and U have the same public API:

```typescript
class T {
  public name: string = "";
  public greet(): void {
    console.log("Hello, " + this.name);
  }
}
class U {
  public name: string = "";
  public greet(): void {
    console.log("Greetings, " + this.name);
  }
}
```

Can we assign a value of type T to a variable of type U?

```typescript
let u: U = new T(); // Is this allowed?
```

Can we pass a value of type T to a function that accepts a parameter of type U?

```typescript
function greeter(u: U) {
  console.log("To " + u.name);
  u.greet();
}
let t: T = new T();
greeter(t); // Is this allowed?
```

In other words, which approach will we take:

- T and U have no inheritance relationship or don't implement the same interface, but since they have the same public API, they are "equal in some sense," so the answer to both questions above is "yes";
- T and U have no inheritance relationship or don't implement the same interface, and should always be considered completely different types, so the answer to both questions above is "no".

Languages that adopt the first approach support structural typing, while languages that adopt the second approach do not support structural typing. Currently, TypeScript supports structural typing, while ArkTS does not.

Whether structural typing helps generate clear, understandable code is not definitively concluded. So why doesn't ArkTS support structural typing?

Because support for structural typing is a major feature that requires extensive consideration and careful implementation in language specifications, compilers, and runtime. Additionally, since ArkTS uses static typing, the runtime needs additional performance overhead to support this feature.

Given this, we currently do not support this feature. Based on actual scenario requirements and feedback, we will reconsider it in the future. For more cases and suggestions, please refer to [Constraint Specifications](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/typescript-to-arkts-migration-guide-V5#约束说明).

## Constraint Specifications

### Object Property Names Must Be Valid Identifiers

**Rule:** arkts-identifiers-as-prop-names

**Level:** Error

In ArkTS, object property names cannot be numbers or strings. Exception: ArkTS supports property names as string literals and string values in enums. Access class properties through property names, and access array elements through numeric indices.

**TypeScript**

```typescript
var x = { name: "x", 2: "3" };
console.log(x["name"]);
console.log(x[2]);
```

**ArkTS**

```typescript
class X {  public name: string = ''}let x: X = { name: 'x' };console.log(x.name);
let y = ['a', 'b', 'c'];console.log(y[2]);
// In scenarios where data needs to be retrieved through non-identifiers (i.e., different types of keys), use Map<Object, some_type>.let z = new Map<Object, string>();z.set('name', '1');z.set(2, '2');console.log(z.get('name'));console.log(z.get(2));
enum Test {  A = 'aaa',  B = 'bbb'}
let obj: Record<string, number> = {  [Test.A]: 1,   // String values in enums  [Test.B]: 2,   // String values in enums  ['value']: 3   // String literals}
```

### Symbol() API Not Supported

**Rule:** arkts-no-symbol

**Level:** Error

The Symbol() API in TypeScript is used to generate unique property names at runtime. Since common use cases for this API don't make sense in statically typed languages, ArkTS doesn't support the Symbol() API. In ArkTS, object layouts are determined at compile time and cannot be changed at runtime.

ArkTS only supports Symbol.iterator.

### Private Fields Starting with # Not Supported

**Rule:** arkts-no-private-identifiers

**Level:** Error

ArkTS doesn't support private fields declared with the # symbol. Use the private keyword instead.

**TypeScript**

```typescript
class C {
  #foo: number = 42;
}
```

**ArkTS**

```typescript
class C {
  private foo: number = 42;
}
```

### Type and Namespace Names Must Be Unique

**Rule:** arkts-unique-names

**Level:** Error

Names of types (classes, interfaces, enums) and namespaces must be unique and different from other names (such as variable names, function names).

**TypeScript**

```typescript
let X: stringtype X = number[] // Type alias has the same name as a variable
```

**ArkTS**

```typescript
let X: stringtype T = number[] // To avoid name conflicts, X is not allowed here
```

### Use let Instead of var

**Rule:** arkts-no-var

**Level:** Error

The let keyword can declare variables in block scope, helping programmers avoid errors. Therefore, ArkTS doesn't support var; use let to declare variables.

**TypeScript**

```typescript
function f(shouldInitialize: boolean) {
  if (shouldInitialize) {
    var x = "b";
  }
  return x;
}
console.log(f(true)); // bconsole.log(f(false)); // undefined
let upperLet = 0;
{
  var scopedVar = 0;
  let scopedLet = 0;
  upperLet = 5;
}
scopedVar = 5; // Visiblescopedlet = 5; // Compile-time error
```

**ArkTS**

```typescript
function f(shouldInitialize: boolean): string {
  let x: string = "a";
  if (shouldInitialize) {
    x = "b";
  }
  return x;
}
console.log(f(true)); // bconsole.log(f(false)); // a
let upperLet = 0;
let scopedVar = 0;
{
  let scopedLet = 0;
  upperLet = 5;
}
scopedVar = 5;
scopedLet = 5; // Compile-time error
```

### Use Specific Types Instead of any or unknown

**Rule:** arkts-no-any-unknown

**Level:** Error

ArkTS doesn't support any and unknown types. Explicitly specify concrete types.

**TypeScript**

```typescript
let value1: anyvalue1 = true;
value1 = 42;
let value2: unknownvalue2 = true;
value2 = 42;
```

**ArkTS**

```typescript
let value_b: boolean = true; // or let value_b = truelet value_n: number = 42; // or let value_n = 42let value_o1: Object = true;let value_o2: Object = 42;
```

### Use class Instead of Types with Call Signatures

**Rule:** arkts-no-call-signatures

**Level:** Error

ArkTS doesn't support call signatures in object types.

**TypeScript**

```typescript
type DescribableFunction = {  description: string  (someArg: string): string // call signature}
function doSomething(fn: DescribableFunction): void {  console.log(fn.description + ' returned ' + fn(''));}
```

**ArkTS**

```typescript
class DescribableFunction {  description: string  public invoke(someArg: string): string {    return someArg;  }  constructor() {    this.description = 'desc';  }}
function doSomething(fn: DescribableFunction): void {  console.log(fn.description + ' returned ' + fn.invoke(''));}
doSomething(new DescribableFunction());
```

### Use class Instead of Types with Constructor Signatures

**Rule:** arkts-no-ctor-signatures-type

**Level:** Error

ArkTS doesn't support constructor signatures in object types. Use classes instead.

**TypeScript**

```typescript
class SomeObject {}
type SomeConstructor = { new (s: string): SomeObject };
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

**ArkTS**

```typescript
class SomeObject {  public f: string  constructor (s: string) {    this.f = s;  }}
function fn(s: string): SomeObject {  return new SomeObject(s);}
```

### Only One Static Block Supported

**Rule:** arkts-no-multiple-static-blocks

**Level:** Error

ArkTS doesn't allow multiple static blocks in a class. If there are multiple static block statements, merge them into one static block.

**TypeScript**

```typescript
class C {
  static s: string;
  static {
    C.s = "aa";
  }
  static {
    C.s = C.s + "bb";
  }
}
```

**ArkTS**

```typescript
class C {  static s: string
  static {    C.s = 'aa'    C.s = C.s + 'bb'  }}
```

**Note**

Static block syntax is currently not supported. After this syntax is supported, using static blocks in .ets files must follow this constraint.

### Index Signatures Not Supported

**Rule:** arkts-no-indexed-signatures

**Level:** Error

ArkTS doesn't allow index signatures; use arrays instead.

**TypeScript**

```typescript
// Interface with index signature:interface StringArray {  [index: number]: string}
function getStringArray(): StringArray {
  return ["a", "b", "c"];
}
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
```

**ArkTS**

```typescript
class X {
  public f: string[] = [];
}
let myArray: X = new X();
const secondItem = myArray.f[1];
```

### Use Inheritance Instead of Intersection Types

**Rule:** arkts-no-intersection-types

**Level:** Error

ArkTS currently doesn't support intersection types; inheritance can be used as an alternative.

**TypeScript**

```typescript
interface Identity {  id: number  name: string}
interface Contact {  email: string  phoneNumber: string}
type Employee = Identity & Contact
```

**ArkTS**

```typescript
interface Identity {  id: number  name: string}
interface Contact {  email: string  phoneNumber: string}
interface Employee extends Identity,  Contact {}
```
