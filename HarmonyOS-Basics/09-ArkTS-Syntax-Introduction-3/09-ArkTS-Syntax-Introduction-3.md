# 09-ArkTS Syntax Introduction (3)

# Classes

Class declarations introduce a new type and define its fields, methods, and constructors.

In the following example, the Person class is defined with fields name and surname, a constructor, and method fullName:

```typescript
class Person {
  name: string = "";
  surname: string = "";
  constructor(n: string, sn: string) {
    this.name = n;
    this.surname = sn;
  }
  fullName(): string {
    return this.name + " " + this.surname;
  }
}
```

After defining a class, you can create instances using the keyword new:

```typescript
let p = new Person("John", "Smith");
console.log(p.fullName());
```

Alternatively, you can create instances using object literals:

```typescript
class Point {
  x: number = 0;
  y: number = 0;
}
let p: Point = { x: 42, y: 42 };
```

## Fields

Fields are variables of a certain type declared directly in a class.

Classes can have instance fields or static fields.

**Instance Fields**

Instance fields exist on each instance of the class. Each instance has its own set of instance fields.

To access instance fields, you need to use an instance of the class.

```typescript
class Person {
  name: string = "";
  age: number = 0;
  constructor(n: string, a: number) {
    this.name = n;
    this.age = a;
  }
  getName(): string {
    return this.name;
  }
}
let p1 = new Person("Alice", 25);
p1.name;
let p2 = new Person("Bob", 28);
p2.getName();
```

**Static Fields**

Fields are declared as static using the keyword static. Static fields belong to the class itself, and all instances of the class share one static field.

To access static fields, you need to use the class name:

```typescript
class Person {
  static numberOfPersons = 0;
  constructor() {
    // ...
    Person.numberOfPersons++;
    // ...
  }
}
Person.numberOfPersons;
```

**Field Initialization**

To reduce runtime errors and achieve better execution performance, ArkTS requires all fields to be explicitly initialized either at declaration or in the constructor. This is the same as the strictPropertyInitialization mode in standard TS.

The following code is illegal in ArkTS:

```typescript
class Person {
  name: string; // undefined
  setName(n: string): void {
    this.name = n;
  }

  getName(): string {
    // Developer uses "string" as return type, which hides the fact that name might be "undefined".
    // A more appropriate approach would be to annotate the return type as "string | undefined" to tell developers all possible return values of this API.
    return this.name;
  }
}
let jack = new Person();
// Assuming no assignment to name in the code, such as calling "jack.setName('Jack')"
jack.getName().length; // Runtime exception: name is undefined
```

In ArkTS, the code should be written as follows:

```typescript
class Person {
  name: string = "";

  setName(n: string): void {
    this.name = n;
  }

  // Type is 'string', cannot be "null" or "undefined"
  getName(): string {
    return this.name;
  }
}
let jack = new Person();
// Assuming no assignment to name in the code, such as calling "jack.setName('Jack')"
jack.getName().length; // 0, no runtime exception
```

The following code shows how to write code if the value of name can be undefined:

```typescript
class Person {
  name?: string; // Might be `undefined`

  setName(n: string): void {
    this.name = n;
  }

  // Compile-time error: name can be "undefined", so the return type of this API cannot be defined only as string type
  getNameWrong(): string {
    return this.name;
  }

  getName(): string | undefined {
    // Return type matches name's type
    return this.name;
  }
}
let jack = new Person();
// Assuming no assignment to name in the code, such as calling "jack.setName('Jack')"

// Compile-time error: Compiler thinks the next line might access properties of undefined, reports error
jack.getName().length; // Compilation fails

jack.getName()?.length; // Compilation succeeds, no runtime error
```

**Getters and Setters**

Setters and getters can be used to provide controlled access to object properties.

In the following example, a setter is used to prevent setting the \_age property to invalid values:

```typescript
class Person {
  name: string = "";
  private _age: number = 0;
  get age(): number {
    return this._age;
  }
  set age(x: number) {
    if (x < 0) {
      throw Error("Invalid age argument");
    }
    this._age = x;
  }
}
let p = new Person();
p.age; // Outputs 0
p.age = -42; // Setting invalid age value throws an error
```

Getters or setters can be defined in classes.

## Methods

Methods belong to classes. Classes can define instance methods or static methods. Static methods belong to the class itself and can only access static fields. Instance methods can access both static fields and instance fields, including private fields of the class.

**Instance Methods**

The following example illustrates how instance methods work.

The calculateArea method calculates the area of a rectangle by multiplying height by width:

```typescript
class RectangleSize {
  private height: number = 0;
  private width: number = 0;
  constructor(height: number, width: number) {
    this.height = height;
    this.width = width;
  }
  calculateArea(): number {
    return this.height * this.width;
  }
}
```

Instance methods must be called through an instance of the class:

```typescript
let square = new RectangleSize(10, 10);
square.calculateArea(); // Output: 100
```

**Static Methods**

Methods are declared as static using the keyword static. Static methods belong to the class itself and can only access static fields.

Static methods define common behavior for the class as a whole.

Static methods must be called through the class name:

```typescript
class Cl {
  static staticMethod(): string {
    return "this is a static method.";
  }
}
console.log(Cl.staticMethod());
```

**Inheritance**

A class can inherit from another class (called the base class) and implement multiple interfaces using the following syntax:

```typescript
class [extends BaseClassName] [implements listOfInterfaces] {
  // ...
}
```

Inheriting classes inherit fields and methods from the base class, but not constructors. Inheriting classes can define new fields and methods, and can also override methods defined by their base class.

The base class is also called the "parent class" or "superclass". The inheriting class is also called the "derived class" or "subclass".

Example:

```typescript
class Person {
  name: string = "";
  private _age = 0;
  get age(): number {
    return this._age;
  }
}
class Employee extends Person {
  salary: number = 0;
  calculateTaxes(): number {
    return this.salary * 0.42;
  }
}
```

Classes containing implements clauses must implement all methods defined in the listed interfaces, except for methods defined with default implementations.

```typescript
interface DateInterface {
  now(): string;
}
class MyDate implements DateInterface {
  now(): string {
    // Implementation here
    return "now";
  }
}
```

**Parent Class Access**

The keyword super can be used to access parent class instance fields, instance methods, and constructors. When implementing subclass functionality, you can obtain the required interface from the parent class through this keyword:

```typescript
class RectangleSize {
  protected height: number = 0;
  protected width: number = 0;

  constructor(h: number, w: number) {
    this.height = h;
    this.width = w;
  }

  draw() {
    /* Draw border */
  }
}
class FilledRectangle extends RectangleSize {
  color = "";
  constructor(h: number, w: number, c: string) {
    super(h, w); // Parent class constructor call
    this.color = c;
  }

  draw() {
    super.draw(); // Parent class method call
    // super.height - can be used here
    /* Fill rectangle */
  }
}
```

**Method Overriding**

Subclasses can override the implementation of methods defined in their parent class. Overridden methods must have the same parameter types as the original method and the same or derived return type.

```typescript
class RectangleSize {
  // ...
  area(): number {
    // Implementation
    return 0;
  }
}
class Square extends RectangleSize {
  private side: number = 0;
  area(): number {
    return this.side * this.side;
  }
}
```

**Method Overload Signatures**

Through overload signatures, different ways to call methods are specified. The specific method is to write multiple method headers with the same name but different signatures for the same method, followed by the method implementation.

```typescript
class C {
  foo(x: number): void;
  /* First signature */
  foo(x: string): void;
  /* Second signature */
  foo(x: number | string): void {
    /* Implementation signature */
  }
}
let c = new C();
c.foo(123); // OK, uses first signature
c.foo("aa"); // OK, uses second signature
```

If two overload signatures have the same name and parameter list, it is an error.

## Constructors

Class declarations can include constructors for initializing object state.

Constructors are defined as follows:

```typescript
constructor ([parameters]) {
  // ...
}
```

If no constructor is defined, a default constructor with an empty parameter list is automatically created, for example:

```typescript
class Point {
  x: number = 0;
  y: number = 0;
}
let p = new Point();
```

In this case, the default constructor initializes fields in the instance using default values of field types.

**Constructors in Derived Classes**

The first statement in a constructor function body can use the keyword super to explicitly call the constructor of the direct parent class.

```typescript
class RectangleSize {
  constructor(width: number, height: number) {
    // ...
  }
}
class Square extends RectangleSize {
  constructor(side: number) {
    super(side, side);
  }
}
```

**Constructor Overload Signatures**

We can specify different ways to call constructors by writing overload signatures. The specific method is to write multiple constructor headers with the same name but different signatures for the same constructor, followed by the constructor implementation.

```typescript
class C {
  constructor(x: number); /* First signature */
  constructor(x: string); /* Second signature */
  constructor(x: number | string) {
    /* Implementation signature */
  }
}
let c1 = new C(123); // OK, uses first signature
let c2 = new C("abc"); // OK, uses second signature
```

If two overload signatures have the same name and parameter list, it is an error.

## Visibility Modifiers

Both methods and properties of classes can use visibility modifiers.

Visibility modifiers include: private, protected, and public. The default visibility is public.

**Public**

public modified class members (fields, methods, constructors) are visible anywhere in the program where the class is accessible.

**Private**

private modified members cannot be accessed outside the class where the member is declared, for example:

```typescript
class C {
  public x: string = "";
  private y: string = "";
  set_y(new_y: string) {
    this.y = new_y; // OK, because y is accessible within the class itself
  }
}
let c = new C();
c.x = "a"; // OK, this field is public
c.y = "b"; // Compile-time error: 'y' is not visible
```

**Protected**

The protected modifier works very similarly to the private modifier, except that protected modified members are allowed to be accessed in derived classes, for example:

```typescript
class Base {
  protected x: string = "";
  private y: string = "";
}
class Derived extends Base {
  foo() {
    this.x = "a"; // OK, accessing protected member
    this.y = "b"; // Compile-time error, 'y' is not visible because it's private
  }
}
```

## Object Literals

Object literals are expressions that can be used to create class instances and provide some initial values. They are more convenient in certain situations and can be used as alternatives to new expressions.

Object literals are represented as: a list of 'property name: value' enclosed in curly braces ({}).

```typescript
class C {
  n: number = 0;
  s: string = "";
}
let c: C = { n: 42, s: "foo" };
```

ArkTS is a statically typed language. As shown in the above example, object literals can only be used in contexts where the type of the literal can be inferred. Other correct examples:

```typescript
class C {
  n: number = 0;
  s: string = "";
}

function foo(c: C) {}
let c: C;
c = { n: 42, s: "foo" }; // Use variable's type
foo({ n: 42, s: "foo" }); // Use parameter's type

function bar(): C {
  return { n: 42, s: "foo" }; // Use return type
}
```

Can also be used in array element types or class field types:

```typescript
class C {
  n: number = 0;
  s: string = "";
}
let cc: C[] = [
  { n: 1, s: "a" },
  { n: 2, s: "b" },
];
```

**Object Literals of Record Type**

Generic Record<K, V> is used to map properties of one type (key type) to another type (value type). Object literals are commonly used to initialize values of this type:

```typescript
let map: Record<string, number> = { John: 25, Mary: 21 };
map["John"]; // 25
```

Type K can be string type or numeric type, while V can be any type.

```typescript
interface PersonInfo {
  age: number;
  salary: number;
}
let map: Record<string, PersonInfo> = {
  John: { age: 25, salary: 10 },
  Mary: { age: 21, salary: 20 },
};
```

## Abstract Classes

Classes with the abstract modifier are called abstract classes. Abstract classes can be used to represent concepts shared by a group of more specific concepts.

If you try to create an instance of an abstract class, a compile-time error occurs:

```typescript
abstract class X {
  field: number;
  constructor(p: number) {
    this.field = p;
  }
}
let x = new X(666); // Compile-time error: Cannot create concrete instance of abstract class
```

Subclasses of abstract classes can be abstract or non-abstract. Non-abstract subclasses of abstract parent classes can be instantiated. Therefore, the constructor of the abstract class and field initializers of non-static fields of the class are executed:

```typescript
abstract class Base {
  field: number;
  constructor(p: number) {
    this.field = p;
  }
}
class Derived extends Base {
  constructor(p: number) {
    super(p);
  }
}
```

**Abstract Methods**

Methods with the abstract modifier are called abstract methods. Abstract methods can be declared but not implemented.

Abstract methods can only exist within abstract classes. If a non-abstract class has abstract methods, a compile-time error occurs:

```typescript
class Y {
  abstract method(p: string); // Compile-time error: Abstract methods can only be within abstract classes.
}
```
