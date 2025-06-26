# 19-ArkTS Common Errors (Part 3)

## arkts-no-obj-literals-as-types

**Application Code**

```typescript
type Person = { name: string; age: number };
```

**Suggested Fix**

```typescript
interface Person {
  name: string;
  age: number;
}
```

## arkts-no-noninferrable-arr-literals

**Application Code**

```typescript
let permissionList = [
  {
    name: "Device Info",
    value:
      "Used to analyze device battery, calls, internet, SIM card issues, etc.",
  },
  { name: "Microphone", value: "Used to add voice when submitting feedback" },
  {
    name: "Storage",
    value: "Used to add local file attachments when submitting feedback",
  },
];
```

**Suggested Fix**

Declare type for object literals

```typescript
class PermissionItem {
  name?: string;
  value?: string;
}
let permissionList: PermissionItem[] = [
  {
    name: "Device Info",
    value:
      "Used to analyze device battery, calls, internet, SIM card issues, etc.",
  },
  { name: "Microphone", value: "Used to add voice when submitting feedback" },
  {
    name: "Storage",
    value: "Used to add local file attachments when submitting feedback",
  },
];
```

## arkts-no-method-reassignment

**Application Code**

```typescript
class C {
  add(left: number, right: number): number {
    return left + right;
  }
}
function sub(left: number, right: number): number {
  return left - right;
}
let c1 = new C();
c1.add = sub;
```

**Suggested Fix**

```typescript
class C {
  add: (left: number, right: number) => number = (
    left: number,
    right: number
  ) => {
    return left + right;
  };
}
function sub(left: number, right: number): number {
  return left - right;
}
let c1 = new C();
c1.add = sub;
```

## arkts-no-polymorphic-unops

**Application Code**

```typescript
let a = +"5";
let b = -"5";
let c = ~"5";
let d = +"string";
```

**Suggested Fix**

```typescript
let a = Number.parseInt("5");
let b = -Number.parseInt("5");
let c = ~Number.parseInt("5");
let d = new Number("string");
```

## arkts-no-type-query

**Application Code**

```typescript
// module1.ts
class C {
  value: number = 0;
}
export let c = new C();
// module2.ts
import { c } from "./module1";
let t: typeof c = { value: 123 };
```

**Suggested Fix**

```typescript
// module1.ts
class C {
  value: number = 0;
}
export { C };
// module2.ts
import { C } from "./module1";
let t: C = { value: 123 };
```

## arkts-no-in

### Use Object.keys to check property existence

**Application Code**

```typescript
function test(str: string, obj: Record<string, Object>) {
  return str in obj;
}
```

**Suggested Fix**

```typescript
function test(str: string, obj: Record<string, Object>) {
  for (let i of Object.keys(obj)) {
    if (i == str) {
      return true;
    }
  }
  return false;
}
```

## arkts-no-destruct-assignment

**Application Code**

```typescript
let map = new Map<string, string>([
  ["a", "a"],
  ["b", "b"],
]);
for (let [key, value] of map) {
  console.log(key);
  console.log(value);
}
```

**Suggested Fix**

Use array

```typescript
let map = new Map<string, string>([
  ["a", "a"],
  ["b", "b"],
]);
for (let arr of map) {
  let key = arr[0];
  let value = arr[1];
  console.log(key);
  console.log(value);
}
```

## arkts-no-types-in-catch

**Application Code**

```typescript
import { BusinessError } from "@kit.BasicServicesKit";
try {
  // ...
} catch (e: BusinessError) {
  console.error(e.message, e.code);
}
```

**Suggested Fix**

```typescript
import { BusinessError } from "@kit.BasicServicesKit";
try {
  // ...
} catch (error) {
  let e: BusinessError = error as BusinessError;
  console.error(e.message, e.code);
}
```

## arkts-no-for-in

**Application Code**

```typescript
interface Person {
  [name: string]: string;
}
let p: Person = {
  name: "tom",
  age: "18",
};
for (let t in p) {
  console.log(p[t]); // log: "tom", "18"
}
```

**Suggested Fix**

```typescript
let p: Record<string, string> = {
  name: "tom",
  age: "18",
};
for (let ele of Object.entries(p)) {
  console.log(ele[1]); // log: "tom", "18"
}
```

## arkts-no-mapped-types

**Application Code**

```typescript
class C {
  a: number = 0;
  b: number = 0;
  c: number = 0;
}
type OptionsFlags = {
  [Property in keyof C]: string;
};
```

**Suggested Fix**

```typescript
class C {
  a: number = 0;
  b: number = 0;
  c: number = 0;
}
type OptionsFlags = Record<keyof C, string>;
```

## arkts-limited-throw

**Application Code**

```typescript
import { BusinessError } from "@kit.BasicServicesKit";
function ThrowError(error: BusinessError) {
  throw error;
}
```

**Suggested Fix**

```typescript
import { BusinessError } from "@kit.BasicServicesKit";
function ThrowError(error: BusinessError) {
  throw error as Error;
}
```

**Reason**

The type of value in throw statement must be Error or its inheritance class. If the inheritance class is a generic, there will be compile-time error. It is recommended to use as to convert the type to Error.

## arkts-no-standalone-this

### Using this in functions

**Application Code**

```typescript
function foo() {
  console.log(this.value);
}
let obj = { value: "abc" };
foo.apply(obj);
```

**Suggested Fix 1**

Use class methods. If this method is used by multiple classes, consider using inheritance mechanism

```typescript
class Test {
  value: string = "";
  constructor(value: string) {
    this.value = value;
  }
  foo() {
    console.log(this.value);
  }
}
let obj: Test = new Test("abc");
obj.foo();
```

**Suggested Fix 2**

Pass this as parameter

```typescript
function foo(obj: Test) {
  console.log(obj.value);
}
class Test {
  value: string = "";
}
let obj: Test = { value: "abc" };
foo(obj);
```

**Suggested Fix 3**

Pass property as parameter

```typescript
function foo(value: string) {
  console.log(value);
}
class Test {
  value: string = "";
}
let obj: Test = { value: "abc" };
foo(obj.value);
```

### Using this in class static methods

**Application Code**

```typescript
class Test {
  static value: number = 123;
  static foo(): number {
    return this.value;
  }
}
```

**Suggested Fix**

```typescript
class Test {
  static value: number = 123;
  static foo(): number {
    return Test.value;
  }
}
```

## arkts-no-spread

**Application Code**

```typescript
// test.d.ets
declare namespace test {
  interface I {
    id: string;
    type: number;
  }
  function foo(): I;
}
export default test;
// app.ets
import test from "test";
let t: test.I = {
  ...test.foo(),
  type: 0,
};
```

**Suggested Fix**

```typescript
// test.d.ets
declare namespace test {
  interface I {
    id: string;
    type: number;
  }
  function foo(): I;
}
export default test;
// app.ets
import test from "test";
let t: test.I = test.foo();
t.type = 0;
```

**Reason**

In ArkTS, object layout is determined at compile time. If you need to spread all properties of one object to assign to another object, it can be done through individual property assignment statements. In this example, the object to be spread and the target object happen to have the same type, so the code can be refactored by changing the object's properties.
