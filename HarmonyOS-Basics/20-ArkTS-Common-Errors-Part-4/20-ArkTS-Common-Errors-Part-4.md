# 20-ArkTS Common Errors (Part 4)

## arkts-no-ctor-signatures-funcs

Declare properties in class instead of on constructor.

**Application Code**

```typescript
class Controller {
  value: string = "";
  constructor(value: string) {
    this.value = value;
  }
}
type ControllerConstructor = new (value: string) => Controller;
class Menu {
  controller: ControllerConstructor = Controller;
  createController() {
    if (this.controller) {
      return new this.controller("abc");
    }
    return null;
  }
}
let t = new Menu();
console.log(t.createController()!.value);
```

**Suggested Fix**

```typescript
class Controller {
  value: string = "";
  constructor(value: string) {
    this.value = value;
  }
}
type ControllerConstructor = () => Controller;
class Menu {
  controller: ControllerConstructor = () => {
    return new Controller("abc");
  };
  createController() {
    if (this.controller) {
      return this.controller();
    }
    return null;
  }
}
let t: Menu = new Menu();
console.log(t.createController()!.value);
```

## arkts-no-globalthis

Since static types cannot be added to globalThis, it can only be accessed through search, causing additional performance overhead. Additionally, types cannot be marked for globalThis properties, making it impossible to ensure safe and high-performance operations on these properties. Therefore, ArkTS does not support globalThis.

1. It is recommended to implement data transfer between different modules through import/export syntax according to business logic.
2. If necessary, you can achieve global object functionality through constructed **singleton objects**. (**Note:** Singleton objects cannot be defined in har. When har is packaged, it will be packaged twice in different haps, making it impossible to achieve singleton.)

**Constructing Singleton Object**

```typescript
// Construct singleton object
export class GlobalContext {
  private constructor() {}
  private static instance: GlobalContext;
  private _objects = new Map<string, Object>();

  public static getContext(): GlobalContext {
    if (!GlobalContext.instance) {
      GlobalContext.instance = new GlobalContext();
    }
    return GlobalContext.instance;
  }

  getObject(value: string): Object | undefined {
    return this._objects.get(value);
  }

  setObject(key: string, objectClass: Object): void {
    this._objects.set(key, objectClass);
  }
}
```

**Application Code**

```typescript
// file1.ts
export class Test {
  value: string = "";
  foo(): void {
    globalThis.value = this.value;
  }
}
// file2.ts
globalThis.value;
```

**Suggested Fix**

```typescript
// file1.ts
import { GlobalContext } from "../GlobalContext";
export class Test {
  value: string = "";
  foo(): void {
    GlobalContext.getContext().setObject("value", this.value);
  }
}
// file2.ts
import { GlobalContext } from "../GlobalContext";
GlobalContext.getContext().getObject("value");
```

## arkts-no-func-apply-bind-call

### Use interfaces from standard library

**Application Code**

```typescript
let arr: number[] = [1, 2, 3, 4];
let str = String.fromCharCode.apply(null, Array.from(arr));
```

**Suggested Fix**

```typescript
let arr: number[] = [1, 2, 3, 4];
let str = String.fromCharCode(...Array.from(arr));
```

### Define methods using bind

**Application Code**

```typescript
class A {
  value: string = "";
  foo: Function = () => {};
}
class Test {
  value: string = "1234";
  obj: A = {
    value: this.value,
    foo: this.foo.bind(this),
  };
  foo() {
    console.log(this.value);
  }
}
```

**Suggested Fix 1**

```typescript
class A {
  value: string = "";
  foo: Function = () => {};
}
class Test {
  value: string = "1234";
  obj: A = {
    value: this.value,
    foo: (): void => this.foo(),
  };
  foo() {
    console.log(this.value);
  }
}
```

**Suggested Fix 2**

```typescript
class A {
  value: string = "";
  foo: Function = () => {};
}
class Test {
  value: string = "1234";
  foo: () => void = () => {
    console.log(this.value);
  };
  obj: A = {
    value: this.value,
    foo: this.foo,
  };
}
```

### Using apply

**Application Code**

```typescript
class A {
  value: string;
  constructor(value: string) {
    this.value = value;
  }
  foo() {
    console.log(this.value);
  }
}
let a1 = new A("1");
let a2 = new A("2");
a1.foo();
a1.foo.apply(a2);
```

**Suggested Fix**

```typescript
class A {
  value: string;
  constructor(value: string) {
    this.value = value;
  }
  foo() {
    this.fooApply(this);
  }
  fooApply(a: A) {
    console.log(a.value);
  }
}
let a1 = new A("1");
let a2 = new A("2");
a1.foo();
a1.fooApply(a2);
```

## arkts-limited-stdlib

### Object.fromEntries()

**Application Code**

```typescript
let entries = new Map([
  ["foo", 123],
  ["bar", 456],
]);
let obj = Object.fromEntries(entries);
```

**Suggested Fix**

```typescript
let entries = new Map([
  ["foo", 123],
  ["bar", 456],
]);
let obj: Record<string, Object> = {};
entries.forEach((value, key) => {
  if (key != undefined && key != null) {
    obj[key] = value;
  }
});
```

### Using Number properties and methods

ArkTS does not allow using global object properties and methods: Infinity, NaN, isFinite, isNaN, parseFloat, parseInt

You can use Number properties and methods: Infinity, NaN, isFinite, isNaN, parseFloat, parseInt

**Application Code**

```typescript
NaN;
isFinite(123);
parseInt("123");
```

**Suggested Fix**

```typescript
Number.NaN;
Number.isFinite(123);
Number.parseInt("123");
```

## arkts-strict-typing(StrictModeError)

### strictPropertyInitialization

**Application Code**

```typescript
interface I {
  name: string;
}
class A {}
class Test {
  a: number;
  b: string;
  c: boolean;
  d: I;
  e: A;
}
```

**Suggested Fix**

```typescript
interface I {
  name: string;
}
class A {}
class Test {
  a: number;
  b: string;
  c: boolean;
  d: I = { name: "abc" };
  e: A | null = null;
  constructor(a: number, b: string, c: boolean) {
    this.a = a;
    this.b = b;
    this.c = c;
  }
}
```

### Type **_ | null is not assignable to type _**

**Application Code**

```typescript
class A {
  bar() {}
}
function foo(n: number) {
  if (n === 0) {
    return null;
  }
  return new A();
}
function getNumber() {
  return 5;
}
let a: A = foo(getNumber());
a.bar();
```

**Suggested Fix**

```typescript
class A {
  bar() {}
}
function foo(n: number) {
  if (n === 0) {
    return null;
  }
  return new A();
}
function getNumber() {
  return 5;
}
let a: A | null = foo(getNumber());
a?.bar();
```

### Strict property initialization check

In class, if a property is not initialized and is not assigned in the constructor, ArkTS will report an error.

**Suggested Fix**

1. Generally, **it is recommended to follow business logic** to initialize properties at declaration or assign values to properties in the constructor. For example:

```typescript
//code with error
class Test {
  value: number;
  flag: boolean;
}
//Method 1, initialize at declaration
class Test {
  value: number = 0;
  flag: boolean = false;
}
//Method 2, assign values in constructor
class Test {
  value: number;
  flag: boolean;
  constructor(value: number, flag: boolean) {
    this.value = value;
    this.flag = flag;
  }
}
```

2. For object types (including function types) A, if you are not sure how to initialize, it is recommended to initialize in one of the following ways:

Method (i) prop: A | null = null

Method (ii) prop?: A

Method (iii) prop： A | undefined = undefined

- From a performance perspective, null type is only used for compile-time type checking and has no impact on virtual machine performance. While undefined | A is considered a union type and may have additional runtime overhead.
- From the perspective of code readability and simplicity, prop?:A is syntactic sugar for prop： A | undefined = undefined, **it is recommended to use optional property syntax**

### Strict function type checking

**Application Code**

```typescript
function foo(fn: (value?: string) => void, value: string): void {}
foo((value: string) => {}, ""); //error
```

**Suggested Fix**

```typescript
function foo(fn: (value?: string) => void, value: string): void {}
foo((value?: string) => {}, "");
```

**Reason**

For example, in the following example, if strict function type checking is not enabled at compile time, this code can compile successfully, but will produce unexpected behavior at runtime. Specifically, in the function body of foo, an undefined is passed to fn (this is acceptable because fn can accept undefined), but at the call site of foo on line 6, the passed (value： string) => { console.log(value.toUpperCase()) } function implementation always treats parameter value as string type, allowing it to call the toUpperCase method. If strict function type checking is not enabled, this code will produce an error of unable to find property on undefined at runtime.

```typescript
function foo(fn: (value?: string) => void, value: string): void {
  let v: string | undefined = undefined;
  fn(v);
}
foo((value: string) => {
  console.log(value.toUpperCase());
}, ""); // Cannot read properties of undefined (reading 'toUpperCase')
```

To avoid unexpected runtime behavior, if strict type checking is enabled at compile time, this code will not compile, thus reminding developers to modify the code to ensure program safety.

### Strict null checking

**Application Code**

```typescript
class Test {
  private value?: string;
  public printValue() {
    console.log(this.value.toLowerCase());
  }
}
let t = new Test();
t.printValue();
```

**Suggested Fix**

When writing code, it is recommended to reduce the use of nullable types. If variables and properties are marked with nullable types, null value checks need to be performed before using them, and different logic should be handled based on whether they are null values.

```typescript
class Test {
  private value?: string;
  public printValue() {
    if (this.value) {
      console.log(this.value.toLowerCase());
    }
  }
}
let t = new Test();
t.printValue();
```

**Reason**

In the first code snippet, if strict null checking is not enabled at compile time, this code can compile successfully, but will produce unexpected behavior at runtime. This is because t's property value is undefined (because value?: string is syntactic sugar for value: string | undefined = undefined). When the printValue method is called on line 11, since this.value's value is not null-checked in the method body and is directly accessed as string type, this causes runtime errors. To avoid unexpected runtime behavior, if strict null checking is enabled at compile time, this code will not compile, thus reminding developers to modify the code (as in the second code snippet) to ensure program safety.
