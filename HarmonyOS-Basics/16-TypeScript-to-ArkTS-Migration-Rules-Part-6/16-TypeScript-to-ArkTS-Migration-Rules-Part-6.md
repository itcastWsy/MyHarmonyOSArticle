# 16-TypeScript to ArkTS Migration Rules (Part 6)

### Enforced Strict Type Checking

**Rule:** arkts-strict-typing

**Level:** Error

During compilation, TypeScript strict mode type checking is performed, including:

noImplicitReturns,

strictFunctionTypes,

strictNullChecks,

strictPropertyInitialization.

**TypeScript**

```typescript
// Only produces compile-time error when noImplicitReturns option is enabled
function foo(s: string): string {
  if (s != "") {
    console.log(s);
    return s;
  } else {
    console.log(s);
  }
}

let n: number = null; // Only produces compile-time error when strictNullChecks option is enabled
```

**ArkTS**

```typescript
function foo(s: string): string {
  console.log(s);
  return s;
}

let n1: number | null = null;
let n2: number = 0;
```

When defining a class, if an instance property cannot be initialized at declaration or in the constructor, you can use the definite assignment assertion operator ! to suppress strictPropertyInitialization errors.

Using the definite assignment assertion operator increases the risk of code errors. Developers must ensure that the instance property is assigned before being used, otherwise runtime exceptions may occur.

Using the definite assignment assertion operator adds runtime type checking, thereby introducing additional runtime overhead, so the use of definite assignment assertion operators should be avoided as much as possible.

Using the definite assignment assertion operator will produce warning: arkts-no-definite-assignment.

**TypeScript**

```typescript
class C {
  name: string; // Only produces compile-time error when strictPropertyInitialization option is enabled
  age: number; // Only produces compile-time error when strictPropertyInitialization option is enabled
}

let c = new C();
```

**ArkTS**

```typescript
class C {
  name: string = "";
  age!: number; // warning: arkts-no-definite-assignment

  initAge(age: number) {
    this.age = age;
  }
}

let c = new C();
c.initAge(10);
```

### No Type Checking Disabled via Comments

**Rule:** arkts-strict-typing-required

**Level:** Error

In ArkTS, type checking is not optional. Disabling type checking via comments is not allowed. @ts-ignore and @ts-nocheck are not supported.

**TypeScript**

```typescript
// @ts-nocheck
// ...
// Code with type checking disabled
// ...

let s1: string = null; // No error

// @ts-ignore
let s2: string = null; // No error
```

**ArkTS**

```typescript
let s1: string | null = null; // No error, proper type
let s2: string = null; // Compile-time error
```

### Allow .ets Files to Import .ets/.ts/.js Source Code, Disallow .ts/.js Files to Import .ets Source Code

**Rule:** arkts-no-ts-deps

**Level:** Error

.ets files can import .ets/.ts/.js source code, but .ts/.js files are not allowed to import .ets source code.

**TypeScript**

```typescript
// app.ets
export class C {
  // ...
}

// lib.ts
import { C } from "app";
```

**ArkTS**

```typescript
// lib1.ets
export class C {
  // ...
}

// lib2.ets
import { C } from "lib1";
```

### Class Cannot Be Used as Object

**Rule:** arkts-no-classes-as-obj

**Level:** Warning

In ArkTS, a class declaration defines a new type, not a value. Therefore, using a class as an object (such as assigning a class to an object) is not supported.

### No Other Statements Before Import Statements

**Rule:** arkts-no-misplaced-imports

**Level:** Error

In ArkTS, except for dynamic import statements, all import statements must be placed before all other statements.

**TypeScript**

```typescript
class C {
  s: string = "";
  n: number = 0;
}

import foo from "module1";
```

**ArkTS**

```typescript
import foo from "module1";

class C {
  s: string = "";
  n: number = 0;
}

import("module2").then(() => {}).catch(() => {}); // Dynamic import
```

### Limited Use of ESObject Type

**Rule:** arkts-limited-esobj

**Level:** Warning

To prevent the abuse of dynamic objects (from .ts/.js files) in static code (.ets files), the use of ESObject type in ArkTS is restricted. The only allowed scenario for using ESObject type is in local variable declarations. ESObject type variable assignments are also restricted and can only be assigned with objects from cross-language calls, such as: ESObject, any, unknown, anonymous type variables. Static type values (defined in .ets files) are prohibited from initializing ESObject type variables. ESObject type variables can only be used in cross-language call functions or assigned to another ESObject type variable.

**ArkTS**

```typescript
// lib.d.ts
declare function foo(): any;
declare function bar(a: any): number;

// main.ets
let e0: ESObject = foo(); // Compile-time error: ESObject type can only be used for local variables

function f() {
  let e1 = foo(); // Compile-time error: e1's type is any
  let e2: ESObject = 1; // Compile-time error: Cannot initialize ESObject type variable with non-dynamic value
  let e3: ESObject = {}; // Compile-time error: Cannot initialize ESObject type variable with non-dynamic value
  let e4: ESObject = []; // Compile-time error: Cannot initialize ESObject type variable with non-dynamic value
  let e5: ESObject = ""; // Compile-time error: Cannot initialize ESObject type variable with non-dynamic value
  e5["prop"]; // Compile-time error: Cannot access properties of ESObject type variable
  e5[1]; // Compile-time error: Cannot access properties of ESObject type variable
  e5.prop; // Compile-time error: Cannot access properties of ESObject type variable

  let e6: ESObject = foo(); // OK, explicitly annotated ESObject type
  let e7 = e6; // OK, assignment using ESObject type
  bar(e7); // OK, ESObject type variable passed to cross-language call function
}
```
