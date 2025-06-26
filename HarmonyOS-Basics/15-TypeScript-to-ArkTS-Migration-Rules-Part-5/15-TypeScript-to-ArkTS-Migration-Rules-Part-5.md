# 15-TypeScript to ArkTS Migration Rules (Part 5)

### No Support for Definite Assignment Assertions

**Rule:** arkts-no-definite-assignment

**Level:** Warning

ArkTS does not support definite assignment assertions, such as: let v!: T. Instead, assign a value to the variable when declaring it.

**TypeScript**

```typescript
let x!: number; // Hint: Initialize x before use
initialize();
function initialize() {
  x = 10;
}
console.log("x = " + x);
```

**ArkTS**

```typescript
function initialize(): number {
  return 10;
}
let x: number = initialize();
console.log("x = " + x);
```

### No Support for Prototype Assignment

**Rule:** arkts-no-prototype-assignment

**Level:** Error

ArkTS does not have the concept of prototypes, so it does not support assignment on prototypes. This feature is not compatible with static typing principles.

**TypeScript**

```typescript
let C = function(p) { this.p = p; // Only produces compile-time error when noImplicitThis option is enabled }
C.prototype = { m() { console.log(this.p); } }
C.prototype.q = function(r: string) { return this.p == r; }
```

**ArkTS**

```typescript
class C {
  p: string = "";
  m() {
    console.log(this.p);
  }
  q(r: string) {
    return this.p == r;
  }
}
```

### No Support for globalThis

**Rule:** arkts-no-globalthis

**Level:** Warning

Since ArkTS does not support dynamically changing object layouts, it does not support global scope and globalThis.

**TypeScript**

```typescript
// In global file
var abc = 100;
// Reference 'abc' from above
let x = globalThis.abc;
```

**ArkTS**

```typescript
// file1
export let abc: number = 100;
// file2
import * as M from "file1";
let x = M.abc;
```

### No Support for Some Utility Types

**Rule:** arkts-no-utility-types

**Level:** Error

ArkTS only supports Partial, Required, Readonly, and Record. It does not support other Utility Types from TypeScript.

For Record type objects, the type of values accessed through index is a union type that includes undefined.

### No Support for Function Property Declaration

**Rule:** arkts-no-func-props

**Level:** Error

Since ArkTS does not support dynamically changing function object layouts, it does not support declaring properties on functions.

### No Support for Function.apply and Function.call

**Rule:** arkts-no-func-apply-call

**Level:** Error

ArkTS does not allow the use of standard library functions Function.apply and Function.call. The standard library uses these functions to explicitly set the this parameter of the called function. In ArkTS, the semantics of this are limited to traditional OOP style, and this is prohibited in function bodies.

### No Support for Function.bind

**Rule:** arkts-no-func-bind

**Level:** Warning

ArkTS does not allow the use of standard library function Function.bind. The standard library uses this function to explicitly set the this parameter of the called function. In ArkTS, the semantics of this are limited to traditional OOP style, and this is prohibited in function bodies.

### No Support for as const Assertions

**Rule:** arkts-no-as-const

**Level:** Error

ArkTS does not support as const assertions. In standard TypeScript, as const is used to annotate literals with their corresponding literal types, while ArkTS does not support literal types.

**TypeScript**

```typescript
// 'hello' type
let x = "hello" as const;
// 'readonly [10, 20]' type
let y = [10, 20] as const;
// '{ readonly text: 'hello' }' type
let z = { text: "hello" } as const;
```

**ArkTS**

```typescript
// 'string' type
let x: string = "hello";
// 'number[]' type
let y: number[] = [10, 20];
class Label {
  text: string = "";
}
// 'Label' type
let z: Label = { text: "hello" };
```

### No Support for Import Assertions

**Rule:** arkts-no-import-assertions

**Level:** Error

Since imports in ArkTS are a compile-time rather than runtime feature, ArkTS does not support import assertions. Runtime verification of the correctness of imported APIs is meaningless for statically typed languages. Use regular import syntax instead.

**TypeScript**

```typescript
import { obj } from "something.json" assert { type: "json" };
```

**ArkTS**

```typescript
// The correctness of importing T will be checked at compile time
import { something } from "module";
```

### Limited Use of Standard Library

**Rule:** arkts-limited-stdlib

**Level:** Error

ArkTS does not allow the use of certain interfaces from the TypeScript or JavaScript standard library. Most interfaces are related to dynamic features. The following interfaces are prohibited in ArkTS:

Global object properties and methods: eval

Object: **proto**, **defineGetter**, **defineSetter**,
**lookupGetter**, **lookupSetter**, assign, create,
defineProperties, defineProperty, freeze,
fromEntries, getOwnPropertyDescriptor, getOwnPropertyDescriptors,
getOwnPropertySymbols, getPrototypeOf,
hasOwnProperty, is, isExtensible, isFrozen,
isPrototypeOf, isSealed, preventExtensions,
propertyIsEnumerable, seal, setPrototypeOf

Reflect: apply, construct, defineProperty, deleteProperty,
getOwnPropertyDescriptor, getPrototypeOf,
isExtensible, preventExtensions,
setPrototypeOf

Proxy: handler.apply(), handler.construct(),
handler.defineProperty(), handler.deleteProperty(), handler.get(),
handler.getOwnPropertyDescriptor(), handler.getPrototypeOf(),
handler.has(), handler.isExtensible(), handler.ownKeys(),
handler.preventExtensions(), handler.set(), handler.setPrototypeOf()
