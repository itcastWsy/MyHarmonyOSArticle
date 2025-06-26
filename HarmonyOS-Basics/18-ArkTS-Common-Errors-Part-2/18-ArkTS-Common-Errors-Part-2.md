# 18-ArkTS Common Errors (Part 2)

## arkts-no-untyped-obj-literals

### Annotating object literal types with class requires class constructor to have no parameters

**Application Code**

```typescript
class Test {
  value: number = 1;
  constructor(value: number) {
    this.value = value;
  }
}
let t: Test = { value: 2 };
```

**Suggested Fix 1**

```typescript
// Remove constructor
class Test {
  value: number = 1;
}
let t: Test = { value: 2 };
```

**Suggested Fix 2**

```typescript
// Use new
class Test {
  value: number = 1;
  constructor(value: number) {
    this.value = value;
  }
}
let t: Test = new Test(2);
```

**Reason**

```typescript
class C {
  value: number = 1;
  constructor(n: number) {
    if (n < 0) {
      throw new Error("Negative");
    }
    this.value = n;
  }
}
let s: C = new C(-2); // Throws exception
let t: C = { value: -2 }; // ArkTS does not support
```

For example, in the above example, if using C to annotate object literal types were allowed, the variable t in the above code would cause behavioral ambiguity. ArkTS prohibits bypassing this behavior through object literals.

### Annotating object literal types with class/interface requires using identifier as object literal key

**Application Code**

```typescript
class Test {
  value: number = 0;
}
let arr: Test[] = [
  {
    value: 1,
  },
  {
    value: 2,
  },
  {
    value: 3,
  },
];
```

**Suggested Fix**

```typescript
class Test {
  value: number = 0;
}
let arr: Test[] = [
  {
    value: 1,
  },
  {
    value: 2,
  },
  {
    value: 3,
  },
];
```

### Annotating object literal types with Record requires using string as object literal key

**Application Code**

```typescript
let obj: Record<string, number | string> = {
  value: 123,
  name: "abc",
};
```

**Suggested Fix**

```typescript
let obj: Record<string, number | string> = {
  value: 123,
  name: "abc",
};
```

### Function parameter type contains index signature

**Application Code**

```typescript
function foo(obj: { [key: string]: string }): string {
  if (obj != undefined && obj != null) {
    return obj.value1 + obj.value2;
  }
  return "";
}
```

**Suggested Fix**

```typescript
function foo(obj: Record<string, string>): string {
  if (obj != undefined && obj != null) {
    return obj.value1 + obj.value2;
  }
  return "";
}
```

### Function argument uses object literal

**Application Code**

```typescript
(fn) => {
  fn({ value: 123, name: "" });
};
```

**Suggested Fix**

```typescript
class T {
  value: number = 0;
  name: string = "";
}
(fn: (v: T) => void) => {
  fn({ value: 123, name: "" });
};
```

### class/interface contains methods

**Application Code**

```typescript
interface T {
  foo(value: number): number;
}
let t: T = {
  foo: (value) => {
    return value;
  },
};
```

**Suggested Fix 1**

```typescript
interface T {
  foo: (value: number) => number;
}
let t: T = {
  foo: (value) => {
    return value;
  },
};
```

**Suggested Fix 2**

```typescript
class T {
  foo: (value: number) => number = (value: number) => {
    return value;
  };
}
let t: T = new T();
```

**Reason**

Methods declared in class/interface should be shared by all instances of the class. ArkTS does not support overriding instance methods through object literals. ArkTS supports function type properties.

### export default object

**Application Code**

```typescript
export default {
  onCreate() {
    // ...
  },
  onDestroy() {
    // ...
  },
};
```

**Suggested Fix**

```typescript
class Test {
  onCreate() {
    // ...
  }
  onDestroy() {
    // ...
  }
}
export default new Test();
```

### Get type through importing namespace

**Application Code**

```typescript
// test.d.ets
declare namespace test {
  interface I {
    id: string;
    type: number;
  }
  function foo(name: string, option: I): void;
}
export default test;
// app.ets
import { test } from "test";
let option = { id: "", type: 0 };
test.foo("", option);
```

**Suggested Fix**

```typescript
// test.d.ets
declare namespace test {
  interface I {
    id: string;
    type: number;
  }
  function foo(name: string, option: I): void;
}
export default test;
// app.ets
import { test } from "test";
let option: test.I = { id: "", type: 0 };
test.foo("", option);
```

**Reason**

The object literal lacks type. Based on test.foo analysis, option's type comes from the declaration file, so we just need to import the type.

Note that in test.d.ets, I is defined in namespace, so in the ets file, first import the namespace, then get the corresponding type through name.

### object literal passed to Object type

**Application Code**

```typescript
function emit(event: string, ...args: Object[]): void {}
emit("", {
  action: 11,
  outers: false,
});
```

**Suggested Fix**

```typescript
function emit(event: string, ...args: Object[]): void {}
let emitArg: Record<string, number | boolean> = {
  action: 11,
  outers: false,
};
emit("", emitArg);
```
