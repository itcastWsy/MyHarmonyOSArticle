# 10-ArkTS Syntax Introduction (4)

# Interfaces

Interface declarations introduce new types. Interfaces are a common way to define code contracts.

Any instance of a class that implements a specific interface can achieve polymorphism through that interface.

Interfaces typically contain declarations of properties and methods.

Example:

```typescript
interface Style {
  color: string; // Property
}
interface AreaSize {
  calculateAreaSize(): number; // Method declaration
  someMethod(): void; // Method declaration
}
```

Example of a class implementing an interface:

```typescript
// Interface:
interface AreaSize {
  calculateAreaSize(): number; // Method declaration
  someMethod(): void; // Method declaration
}

// Implementation:
class RectangleSize implements AreaSize {
  private width: number = 0;
  private height: number = 0;
  someMethod(): void {
    console.log("someMethod called");
  }
  calculateAreaSize(): number {
    this.someMethod(); // Call another method and return result
    return this.width * this.height;
  }
}
```

## Interface Properties

Interface properties can be in the form of fields, getters, setters, or combinations of getters and setters.

Property fields are just a convenient notation for getter/setter pairs. The following expressions are equivalent:

```typescript
interface Style {
  color: string;
}
interface Style {
  get color(): string;
  set color(x: string);
}
```

Classes implementing interfaces can also use the following two approaches:

```typescript
interface Style {
  color: string;
}
class StyledRectangle implements Style {
  color: string = "";
}
interface Style {
  color: string;
}
class StyledRectangle implements Style {
  private _color: string = "";
  get color(): string {
    return this._color;
  }
  set color(x: string) {
    this._color = x;
  }
}
```

## Interface Inheritance

Interfaces can inherit from other interfaces, as shown in the following example:

```typescript
interface Style {
  color: string;
}
interface ExtendedStyle extends Style {
  width: number;
}
```

Inheriting interfaces contain all properties and methods of the inherited interface, and can also add their own properties and methods.

## Abstract Classes and Interfaces

Both abstract classes and interfaces cannot be instantiated. Abstract classes are abstractions of classes, used to capture common characteristics of subclasses, while interfaces are abstractions of behavior. In ArkTS, the differences between abstract classes and interfaces are as follows:

- A class can only inherit from one abstract class, while a class can implement one or more interfaces;
- Interfaces cannot contain static code blocks and static methods, while abstract classes can have static code blocks and static methods;
- Abstract classes can contain method implementations, but interfaces are completely abstract with no method implementations;
- Abstract classes can have constructors, while interfaces cannot have constructors.

# Generic Types and Functions

Generic types and functions allow creating code that operates on various types, not just supporting a single type.

## Generic Classes and Interfaces

Classes and interfaces can be defined as generic, adding parameters to type definitions, such as the type parameter Element in the following example:

```typescript
class CustomStack<Element> {
  public push(e: Element): void {
    // ...
  }
}
```

To use type CustomStack, you must specify type arguments for each type parameter:

```typescript
let s = new CustomStack<string>();
s.push("hello");
```

The compiler ensures type safety when using generic types and functions. See the following example:

```typescript
let s = new CustomStack<string>();
s.push(55); // Will cause compile-time error
```

## Generic Constraints

Type parameters of generic types can be constrained to only take certain specific values. For example, the Key type parameter in the MyHashMap<Key, Value> class must have a hash method.

```typescript
interface Hashable {
  hash(): number;
}
class MyHashMap<Key extends Hashable, Value> {
  public set(k: Key, v: Value) {
    let h = k.hash();
    // ...other code...
  }
}
```

In the above example, the Key type extends Hashable, and all methods of the Hashable interface can be called for the key.

## Generic Functions

Using generic functions allows writing more general code. For example, a function that returns the last element of an array:

```typescript
function last(x: number[]): number {
  return x[x.length - 1];
}
last([1, 2, 3]); // 3
```

If you need to define the same function for any array, use type parameters to define the function as generic:

```typescript
function last<T>(x: T[]): T {
  return x[x.length - 1];
}
```

Now, this function can be used with any array.

In function calls, type arguments can be set explicitly or implicitly:

```typescript
// Explicitly set type arguments
last<string>(["aa", "bb"]);
last<number>([1, 2, 3]);

// Implicitly set type arguments
// Compiler determines type arguments based on the types of call parameters
last([1, 2, 3]);
```

## Generic Default Values

Type parameters of generic types can have default values. This allows using only the generic type name without specifying actual type arguments. The following example demonstrates this for both classes and functions.

```typescript
class SomeType {}
interface Interface<T1 = SomeType> {}
class Base<T2 = SomeType> {}
class Derived1 extends Base implements Interface {}
// Derived1 is semantically equivalent to Derived2
class Derived2 extends Base<SomeType> implements Interface<SomeType> {}

function foo<T = number>(): T {
  // ...
}
foo();
// This function is semantically equivalent to the following call
foo<number>();
```

# Null Safety

By default, all types in ArkTS are non-nullable, so type values cannot be null. This is similar to TypeScript's strict null checks mode (strictNullChecks), but with stricter rules.

In the following example, all lines will cause compile-time errors:

```typescript
let x: number = null; // Compile-time error
let y: string = null; // Compile-time error
let z: number[] = null; // Compile-time error
```

Variables that can be null are defined as union type T | null.

```typescript
let x: number | null = null;
x = 1; // ok
x = null; // ok
if (x != null) {
  /* do something */
}
```

## Non-null Assertion Operator

The postfix operator ! can be used to assert that its operand is non-null.

When applied to values of nullable types, its compile-time type becomes non-nullable. For example, the type changes from T | null to T:

```typescript
class A {
  value: number = 0;
}

function foo(a: A | null) {
  a.value; // Compile-time error: Cannot access properties of nullable value
  a!.value; // Compilation passes, if a is non-null at runtime, can access a's properties; if a is null at runtime, runtime exception occurs
}
```

## Nullish Coalescing Operator

The nullish coalescing binary operator ?? is used to check if the evaluation of the left expression equals null or undefined. If so, the result of the expression is the right expression; otherwise, the result is the left expression.

In other words, a ?? b is equivalent to the ternary operator (a != null && a != undefined) ? a : b.

In the following example, the getNick method returns the nickname if set; otherwise, returns an empty string:

```typescript
class Person {
  // ...
  nick: string | null = null;
  getNick(): string {
    return this.nick ?? "";
  }
}
```

## Optional Chaining

When accessing object properties, if the property is undefined or null, the optional chaining operator returns undefined.

```typescript
class Person {
  nick: string | null = null;
  spouse?: Person;
  setSpouse(spouse: Person): void {
    this.spouse = spouse;
  }
  getSpouseNick(): string | null | undefined {
    return this.spouse?.nick;
  }
  constructor(nick: string) {
    this.nick = nick;
    this.spouse = undefined;
  }
}
```

**Note**: The return type of getSpouseNick must be string | null | undefined, because this method might return null or undefined.

Optional chaining can be arbitrarily long and can contain any number of ?. operators.

In the following example, if a Person instance has a non-null spouse property, and spouse has a non-null nick property, then spouse.nick is output. Otherwise, undefined is output:

```typescript
class Person {
  nick: string | null = null;
  spouse?: Person;
  constructor(nick: string) {
    this.nick = nick;
    this.spouse = undefined;
  }
}
let p: Person = new Person("Alice");
p.spouse?.nick; // undefined
```

# Modules

Programs can be divided into multiple sets of compilation units or modules.

Each module has its own scope, meaning any declarations (variables, functions, classes, etc.) created in a module are not visible outside that module unless they are explicitly exported.

Conversely, variables, functions, classes, interfaces, etc. exported from another module must first be imported into the module.

## Export

Top-level declarations can be exported using the keyword export.

Unexported declaration names are considered private names and can only be used within the module where the name is declared.

```typescript
export class Point {
  x: number = 0;
  y: number = 0;
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}
export let Origin = new Point(0, 0);
export function Distance(p1: Point, p2: Point): number {
  return Math.sqrt(
    (p2.x - p1.x) * (p2.x - p1.x) + (p2.y - p1.y) * (p2.y - p1.y)
  );
}
```

## Import

**Static Import**

Import declarations are used to import entities exported from other modules and provide their bindings in the current module. Import declarations consist of two parts:

- Import path, used to specify the imported module;
- Import binding, used to define the set of available entities in the imported module and their usage form (qualified or unqualified usage).

Import bindings can have several forms.

Assume a module has path "./utils" and exported entities "X" and "Y".

Import binding \* as A means binding name "A", through which all entities exported from the module specified by the import path can be accessed via A.name:

```typescript
import * as Utils from "./utils";
Utils.X; // Represents X from Utils
Utils.Y; // Represents Y from Utils
```

Import binding { ident1, ..., identN } means binding exported entities with specified names, which can be used as simple names:

```typescript
import { X, Y } from "./utils";
X; // Represents X from utils
Y; // Represents Y from utils
```

If the identifier list defines ident as alias, then entity ident will be bound under the name alias:

```typescript
import { X as Z, Y } from "./utils";
Z; // Represents X from Utils
Y; // Represents Y from Utils
X; // Compile-time error: 'X' is not visible
```

**Dynamic Import**

In some scenarios of application development, if you want to import modules conditionally or on-demand, you can use dynamic import instead of static import.

The import() syntax, commonly called dynamic import, is a function-like expression used to dynamically import modules. When called this way, it returns a promise.

As shown in the following example, import(modulePath) can load a module and return a promise that resolves to a module object containing all its exports. This expression can be called anywhere in the code.

```typescript
// Calc.ts
export function add(a: number, b: number): number {
  let c = a + b;
  console.info("Dynamic import, %d + %d = %d", a, b, c);
  return c;
}

// Index.ts
import("./Calc")
  .then((obj: ESObject) => {
    console.info(obj.add(3, 5));
  })
  .catch((err: Error) => {
    console.error("Module dynamic import error: ", err);
  });
```

If in an async function, you can use let module = await import(modulePath).

```typescript
// say.ts
export function hi() {
  console.log("Hello");
}
export function bye() {
  console.log("Bye");
}
```

Then, you can perform dynamic import like this:

```typescript
async function test() {
  let ns = await import("./say");
  let hi = ns.hi;
  let bye = ns.bye;
  hi();
  bye();
}
```

For more business scenarios and usage examples of dynamic import, see [Dynamic import](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-dynamic-import-V5).

**Importing HarmonyOS SDK Open Capabilities**

Open capabilities (interfaces) provided by HarmonyOS SDK also need to be used after import declarations. You can directly import interface modules to use all interface capabilities within that module, for example:

```typescript
import UIAbility from "@ohos.app.ability.UIAbility";
```

Starting from HarmonyOS NEXT Developer Preview 1, the Kit concept is introduced. The SDK encapsulates interface modules under the same Kit, and developers can use the interface capabilities contained in a Kit by importing the Kit in sample code. The interface modules encapsulated by Kits can be viewed in the definitions of each Kit in the Kit subdirectory under the SDK directory.

There are three ways to use open capabilities by importing Kits:

- Method 1: Import interface capabilities of a single module under a Kit. For example:

  ```typescript
  import { UIAbility } from "@kit.AbilityKit";
  ```

- Method 2: Import interface capabilities of multiple modules under a Kit. For example:

  ```typescript
  import { UIAbility, Ability, Context } from "@kit.AbilityKit";
  ```

- Method 3: Import interface capabilities of all modules contained in a Kit. For example:

  ```typescript
  import * as module from "@kit.AbilityKit";
  ```

  Where "module" is an alias that can be customized, and then module interfaces are called through this name.

  Note

  Method 3 may import too many unused modules, causing the compiled HAP package to be too large and occupy too many resources. Please use with caution.

## Top-level Statements

Top-level statements are statements written directly at the outermost level of a module, not wrapped in any functions, classes, or block scopes. Top-level statements include variable declarations, function declarations, expressions, etc.

# Keywords

## this

The keyword this can only be used in instance methods of classes.

**Example**

```typescript
class A {
  count: string = "a";
  m(i: string): void {
    this.count = i;
  }
}
```

Usage restrictions:

- this type is not supported
- Using this in functions and static methods of classes is not supported

**Example**

```typescript
class A {
  n: number = 0;
  f1(arg1: this) {} // Compile-time error, this type not supported
  static f2(arg1: number) {
    this.n = arg1; // Compile-time error, using this in static methods of classes not supported
  }
}

function foo(arg1: number) {
  this.n = i; // Compile-time error, using this in functions not supported
}
```

The keyword this points to:

- The object that calls the instance method
- The object being constructed
