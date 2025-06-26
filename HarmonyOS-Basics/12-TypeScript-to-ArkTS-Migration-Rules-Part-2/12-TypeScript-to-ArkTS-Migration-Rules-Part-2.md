# 12-TypeScript to ArkTS Migration Rules (Part 2)

### This type is not supported

**Rule:** arkts-no-typing-with-this

**Level:** Error

ArkTS does not support this type. Use explicit concrete types instead.

**TypeScript**

```typescript
interface ListItem {  getHead(): this}
class C {  n: number = 0
  m(c: this) {    // ...  }}
```

**ArkTS**

```typescript
interface ListItem {  getHead(): ListItem}
class C {  n: number = 0
  m(c: C) {    // ...  }}
```

### Conditional types are not supported

**Rule:** arkts-no-conditional-types

**Level:** Error

ArkTS does not support conditional type aliases. Introduce new types with explicit constraints or rewrite the logic using Object.

The infer keyword is not supported.

**TypeScript**

```typescript
type X<T> = T extends number ? T: nevertype Y<T> = T extends Array<infer Item> ? Item: never
```

**ArkTS**

```typescript
// Provide explicit constraints in type aliasestype X1<T extends number> = T
// Rewrite with Object, less type control, requires more type checking for safetytype X2<T> = Object
// Item must be used as a generic parameter and can be instantiated correctlytype YI<Item, T extends Array<Item>> = Item
```

### Field declaration in constructor is not supported

**Rule:** arkts-no-ctor-prop-decls

**Level:** Error

ArkTS does not support declaring class fields in the constructor. Declare these fields in the class.

**TypeScript**

```typescript
class Person {
  constructor(
    protected ssn: string,
    private firstName: string,
    private lastName: string
  ) {
    this.ssn = ssn;
    this.firstName = firstName;
    this.lastName = lastName;
  }
  getFullName(): string {
    return this.firstName + " " + this.lastName;
  }
}
```

**ArkTS**

```typescript
class Person {  protected ssn: string  private firstName: string  private lastName: string
  constructor(ssn: string, firstName: string, lastName: string) {    this.ssn = ssn;    this.firstName = firstName;    this.lastName = lastName;  }
  getFullName(): string {    return this.firstName + ' ' + this.lastName;  }}
```

### Constructor signatures in interfaces are not supported

**Rule:** arkts-no-ctor-signatures-iface

**Level:** Error

ArkTS does not support constructor signatures in interfaces. Use functions or methods instead.

**TypeScript**

```typescript
interface I {
  new (s: string): I;
}
function fn(i: I) {
  return new i("hello");
}
```

**ArkTS**

```typescript
interface I {
  create(s: string): I;
}
function fn(i: I) {
  return i.create("hello");
}
```

### Indexed access types are not supported

**Rule:** arkts-no-aliases-by-index

**Level:** Error

ArkTS does not support indexed access types.

### Accessing fields by index is not supported

**Rule:** arkts-no-props-by-index

**Level:** Error

ArkTS does not support dynamic field declaration or dynamic field access. You can only access fields that are declared in the class or inherited visible fields. Accessing other fields will cause compile-time errors.

Use dot notation to access fields (e.g., obj.field). Index access (obj[field]) is not supported.

ArkTS supports accessing elements in TypedArray (e.g., Int32Array) by index.

**TypeScript**

```typescript
class Point {  x: string = ''  y: string = ''}let p: Point = {x: '1', y: '2'};console.log(p['x']);
class Person {  name: string = ''  age: number = 0;  [key: string]: string | number}
let person: Person = {  name: 'John',  age: 30,  email: '***@example.com',  phoneNumber: '18*********',}
```

**ArkTS**

```typescript
class Point {  x: string = ''  y: string = ''}let p: Point = {x: '1', y: '2'};console.log(p.x);
class Person {  name: string  age: number  email: string  phoneNumber: string
  constructor(name: string, age: number, email: string,        phoneNumber: string) {    this.name = name;    this.age = age;    this.email = email;    this.phoneNumber = phoneNumber;  }}
let person = new Person('John', 30, '***@example.com', '18*********');console.log(person['name']);     // Compile-time errorconsole.log(person.unknownProperty); // Compile-time error
let arr = new Int32Array(1);arr[0];
```

### Structural typing is not supported

**Rule:** arkts-no-structural-typing

**Level:** Error

ArkTS does not support structural typing. The compiler cannot compare the public APIs of two types and decide if they are equivalent. Use other mechanisms such as inheritance, interfaces, or type aliases.

**TypeScript**

```typescript
interface I1 {  f(): string}
interface I2 { // I2 is equivalent to I1  f(): string}
class X {  n: number = 0  s: string = ''}
class Y { // Y is equivalent to X  n: number = 0  s: string = ''}
let x = new X();let y = new Y();
console.log('Assign X to Y');y = x;
console.log('Assign Y to X');x = y;
function foo(x: X) {  console.log(x.n + x.s);}
// Since X and Y have equivalent APIs, X and Y are equivalentfoo(new X());foo(new Y());
```

**ArkTS**

```typescript
interface I1 {  f(): string}
type I2 = I1 // I2 is an alias of I1
class B {  n: number = 0  s: string = ''}
// D inherits from B, establishing a subtype-supertype relationshipclass D extends B {  constructor() {    super()  }}
let b = new B();let d = new D();
console.log('Assign D to B');b = d; // Legal assignment because B is the parent class of D
// Assigning b to d will cause a compile-time error// d = b
interface Z {   n: number   s: string}
// Class X implements interface Z, establishing a relationship between X and Zclass X implements Z {  n: number = 0  s: string = ''}
// Class Y implements interface Z, establishing a relationship between Y and Zclass Y implements Z {  n: number = 0  s: string = ''}
let x: Z = new X();let y: Z = new Y();
console.log('Assign X to Y');y = x // Legal assignment, they are the same type
console.log('Assign Y to X');x = y // Legal assignment, they are the same type
function foo(c: Z): void {  console.log(c.n + c.s);}
// Classes X and Y implement the same interface, so both function calls are legalfoo(new X());foo(new Y());
```

### Explicit type arguments are required for generic functions

**Rule:** arkts-no-inferred-generic-params

**Level:** Error

ArkTS allows omitting generic type arguments if the concrete type can be inferred from the parameters passed to the generic function. Otherwise, omitting generic type arguments will cause a compile-time error.

Inferring generic type parameters based solely on the return type of a generic function is prohibited.

**TypeScript**

```typescript
function choose<T>(x: T, y: T): T {
  return Math.random() < 0.5 ? x : y;
}
let x = choose(10, 20); // Inferred as choose<number>(...)let y = choose('10', 20); // Compile-time error
function greet<T>(): T {
  return "Hello" as T;
}
let z = greet(); // T is inferred as "unknown"
```

**ArkTS**

```typescript
function choose<T>(x: T, y: T): T {
  return Math.random() < 0.5 ? x : y;
}
let x = choose(10, 20); // Inferred as choose<number>(...)let y = choose('10', 20); // Compile-time error
function greet<T>(): T {
  return "Hello" as T;
}
let z = greet<string>();
```

### Explicit type annotation is required for object literals

**Rule:** arkts-no-untyped-obj-literals

**Level:** Error

In ArkTS, explicit type annotation is required for object literals. Otherwise, a compile-time error will occur. In some scenarios, the compiler can infer the type of the literal from the context.

Using literals to initialize classes and interfaces is not supported in the following contexts:

- Initializing any object with any, Object, or object type
- Initializing classes or interfaces with methods
- Initializing classes that contain custom constructors with parameters
- Initializing classes with readonly fields

**Example 1**

**TypeScript**

```typescript
let o1 = { n: 42, s: "foo" };
let o2: Object = { n: 42, s: "foo" };
let o3: object = { n: 42, s: "foo" };
let oo: Object[] = [
  { n: 1, s: "1" },
  { n: 2, s: "2" },
];
```

**ArkTS**

```typescript
class C1 {  n: number = 0  s: string = ''}
let o1: C1 = {n: 42, s: 'foo'};let o2: C1 = {n: 42, s: 'foo'};let o3: C1 = {n: 42, s: 'foo'};
let oo: C1[] = [{n: 1, s: '1'}, {n: 2, s: '2'}];
```

**Example 2**

**TypeScript**

```typescript
class C2 {  s: string  constructor(s: string) {    this.s = 's =' + s;  }}let o4: C2 = {s: 'foo'};
```

**ArkTS**

```typescript
class C2 {  s: string  constructor(s: string) {    this.s = 's =' + s;  }}let o4 = new C2('foo');
```

**Example 3**

**TypeScript**

```typescript
class C3 {  readonly n: number = 0  readonly s: string = ''}let o5: C3 = {n: 42, s: 'foo'};
```

**ArkTS**

```typescript
class C3 {  n: number = 0  s: string = ''}let o5: C3 = {n: 42, s: 'foo'};
```

**Example 4**

**TypeScript**

```typescript
abstract class A {}
let o6: A = {};
```

**ArkTS**

```typescript
abstract class A {}
class C extends A {}
let o6: C = {}; // or let o6: C = new C()
```

**Example 5**

**TypeScript**

```typescript
class C4 {  n: number = 0  s: string = ''  f() {    console.log('Hello');  }}let o7: C4 = {n: 42, s: 'foo', f: () => {}};
```

**ArkTS**

```typescript
class C4 {  n: number = 0  s: string = ''  f() {    console.log('Hello');  }}let o7 = new C4();o7.n = 42;o7.s = 'foo';
```

**Example 6**

**TypeScript**

```typescript
class Point {  x: number = 0  y: number = 0}
function getPoint(o: Point): Point {  return o;}
// TS supports structural typing, can infer p's type as Pointlet p = {x: 5, y: 10};getPoint(p);
// Can infer the object literal's type as Point from contextgetPoint({x: 5, y: 10});
```

**ArkTS**

```typescript
class Point {  x: number = 0  y: number = 0
  // Before literal initialization, use constructor() to create a valid object.  // Since no constructor is defined for Point, the compiler will automatically add a default constructor.}
function getPoint(o: Point): Point {  return o;}
// Literal initialization requires explicit type definitionlet p: Point = {x: 5, y: 10};getPoint(p);
// getPoint accepts Point type, literal initialization creates a new instance of PointgetPoint({x: 5, y: 10});
```

### Object literals cannot be used for type declarations

**Rule:** arkts-no-obj-literals-as-types

**Level:** Error

ArkTS does not support using object literals to declare types. Use classes or interfaces to declare types.

**TypeScript**

```typescript
let o: { x: number; y: number } = { x: 2, y: 3 };
type S = Set<{ x: number; y: number }>;
```

**ArkTS**

```typescript
class O {  x: number = 0  y: number = 0}
let o: O = {x: 2, y: 3};
type S = Set<O>
```

### Array literals must contain only elements of inferrable types

**Rule:** arkts-no-noninferrable-arr-literals

**Level:** Error

Essentially, ArkTS infers the type of an array literal as the union type of all elements in the array. If any element's type cannot be inferred from the context (e.g., untyped object literals), a compile-time error will occur.

**TypeScript**

```typescript
let a = [
  { n: 1, s: "1" },
  { n: 2, s: "2" },
];
```

**ArkTS**

```typescript
class C {  n: number = 0  s: string = ''}
let a1 = [{n: 1, s: '1'} as C, {n: 2, s: '2'} as C]; // a1's type is "C[]"let a2: C[] = [{n: 1, s: '1'}, {n: 2, s: '2'}];    // a2's type is "C[]"
```

### Use arrow functions instead of function expressions

**Rule:** arkts-no-func-expressions

**Level:** Error

ArkTS does not support function expressions. Use arrow functions.

**TypeScript**

```typescript
let f = function (s: string) {
  console.log(s);
};
```

**ArkTS**

```typescript
let f = (s: string) => {
  console.log(s);
};
```

### Class expressions are not supported

**Rule:** arkts-no-class-literals

**Level:** Error

ArkTS does not support class expressions. A class must be explicitly declared.

**TypeScript**

```typescript
const Rectangle = class {  constructor(height: number, width: number) {    this.height = height;    this.width = width;  }
  height  width}
const rectangle = new Rectangle(0.0, 0.0);
```

**ArkTS**

```typescript
class Rectangle {  constructor(height: number, width: number) {    this.height = height;    this.width = width;  }
  height: number  width: number}
const rectangle = new Rectangle(0.0, 0.0);
```

### Classes cannot be implemented

**Rule:** arkts-implements-only-iface

**Level:** Error

ArkTS does not allow classes to be implemented. Only interfaces can be implemented.

**TypeScript**

```typescript
class C {
  foo() {}
}
class C1 implements C {
  foo() {}
}
```

**ArkTS**

```typescript
interface C {
  foo(): void;
}
class C1 implements C {
  foo() {}
}
```

### Modifying object methods is not supported

**Rule:** arkts-no-method-reassignment

**Level:** Error

ArkTS does not support modifying object methods. In static languages, object layout is determined. All object instances of a class share the same methods.

If you need to add methods to a specific object, you can encapsulate functions or use inheritance mechanisms.

**TypeScript**

```typescript
class C {
  foo() {
    console.log("foo");
  }
}
function bar() {
  console.log("bar");
}
let c1 = new C();
let c2 = new C();
c2.foo = bar;
c1.foo(); // fooc2.foo(); // bar
```

**ArkTS**

```typescript
class C {
  foo() {
    console.log("foo");
  }
}
class Derived extends C {
  foo() {
    console.log("Extra");
    super.foo();
  }
}
function bar() {
  console.log("bar");
}
let c1 = new C();
let c2 = new C();
c1.foo(); // fooc2.foo(); // foo
let c3 = new Derived();
c3.foo(); // Extra foo
```
