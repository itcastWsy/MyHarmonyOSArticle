# HarmonyOS Next V2 State Management: AppStorageV2 and PersistenceV2

## Preface

During the development of `HarmonyOS` applications, we have learned quite a lot about state management related technologies, such as:

- `@ObservedV2` decorator and `@Trace` decorator: Class property change observation
- `@ComponentV2` decorator: Custom components
- `@Local` decorator: Component internal state
- `@Param`: Component external input
- `@Once`: Initialize synchronization once
- `@Event` decorator: Component output
- `@Monitor` decorator: State variable modification monitoring
- `@Provider` decorator and `@Consumer` decorator: Cross-component hierarchical bidirectional synchronization (not covered yet)
- `@Computed` decorator: Computed properties
- `@Type` decorator: Mark class property types

All the above state management technologies revolve around the component's internal self. The `AppStorageV2` and `PersistenceV2` to be explained now can be understood as application/global state management technologies.

1. `AppStorageV2` is application-level data management technology, cross-component and cross-page. All UIAbility instances within the main thread can share data. However, data will be automatically destroyed when exiting the application.

2. `PersistenceV2` is application-level data persistence technology. Data is directly stored on the device disk, and after exiting and re-entering, the data still exists.

## AppStorageV2

In actual development, we inevitably need to share data in real-time across multiple pages or components, such as personal information. In such cases, we can consider storing data in `AppStorageV2`.

`AppStorageV2` is application-level data management technology, cross-component and cross-page. All UIAbility instances within the main thread can share data. However, data will be automatically destroyed when exiting the application.

![image-20240807232620032](readme.assets/image-20240807232620032.png)

## AppStorageV2 Core APIs

| Name    | Function                             |
| ------- | ------------------------------------ |
| connect | Create or retrieve stored data       |
| remove  | Delete stored data for specified key |
| keys    | Return all keys in AppStorageV2      |

### connect

Create or retrieve stored data

| connect      | Description                                                                                                                                                                              |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Parameters   | type: Specified type. If key is not specified, use type.name as key;<br />keyOrDefaultCreater: Specified key or default data constructor;<br />defaultCreator: Default data constructor. |
| Return Value | When creating or retrieving data successfully, return data; otherwise return undefined.                                                                                                  |

**Example Code**:

**Index.ets**

```typescript
import { AppStorageV2, router } from '@kit.ArkUI'

@ObservedV2
export class Person {
  @Trace
  age: number = 10
}

@Entry
@ComponentV2
struct Index {
  @Local person: Person = AppStorageV2.connect(Person, () => new Person)!

  build() {
    Column() {
      Text("Page A")
      // Preview may not work
      Button(`Age: ${this.person.age}`)
        .onClick(() => {
          this.person.age++
        })

      Button("Navigate to Page B")
        .onClick(() => {
          router.pushUrl({
            url: "pages/Index2"
          })
        })
    }
  }
}
```

**Code Explanation**:

`@Local person: Person = AppStorageV2.connect(Person, () => new Person)!` creates or reads data with `key` as `Person`.

`@Local` is used to decorate person, indicating that person is a state, and when the state changes, it will cause UI updates.

`.connect(Person,..)` `Person`, as the first parameter of the `connect` method, represents the `Person` type.

`.connect(Person, () => new Person)!`, since no specific `key` is passed in `connect`, `Person.name` is treated as the `key`. This is equivalent to:

`.connect(Person, Person.name,() => new Person)!` or

`.connect(Person,'Person',() => new Person)!`

![image-20240807234518067](readme.assets/image-20240807234518067.png)

At the same time, because of the effect of `() => new Person`, the initial data is also available.

Finally, because the return value of `connect` might be null, a `!` is added at the end to indicate non-null assertion.

**It's worth noting** that the above code will show `age = undefined` when displayed in the preview, so you need to test on a simulator.

![image-20240807234838624](readme.assets/image-20240807234838624.png)

Finally, if you use the above code on Page A and modify the age, then Page B also uses age, you can see that the data on both pages is the same.

**Index2.ets**

```typescript
import { Person } from "./Index";
import { AppStorageV2 } from '@kit.ArkUI';

@Entry
@ComponentV2
struct Index2 {
  @Local person: Person = AppStorageV2.connect(Person, () => new Person)!

  build() {
    Column() {
      Text("Page B")
      Button(`Age: ${this.person.age}`)
    }
  }
}
```

**Result**

![PixPin_2024-08-07_23-55-31](readme.assets/PixPin_2024-08-07_23-55-31.gif)

### remove

Delete stored data for specified key

| remove       | Description                                                                                         |
| ------------ | --------------------------------------------------------------------------------------------------- |
| Parameters   | keyOrType: The key to be deleted; if a type is specified, the key to be deleted is the type's name. |
| Return Value | None.                                                                                               |

For example: `AppStorageV2.remove(Person)` is actually equivalent to

`AppStorageV2.remove(Person.name)` or `AppStorageV2.remove('Person')`

**Example Code:**

```typescript
import { AppStorageV2, router } from '@kit.ArkUI'

@ObservedV2
export class Person {
  @Trace
  age: number = 10
}

@Entry
@ComponentV2
struct Index {
  @Local person: Person = AppStorageV2.connect(Person, () => new Person)!

  build() {
    Column({ space: 10 }) {
      Text("Page A")
      // Preview may not work
      Button(`Age: ${this.person.age}`)
        .onClick(() => {
          this.person.age++
        })

      Button("Navigate to Page B")
        .onClick(() => {
          router.pushUrl({
            url: "pages/Index2"
          })
        })

      Button("Delete Data")
        .onClick(() => {
          AppStorageV2.remove(Person)
        })
    }
  }
}
```

If the `remove` method is executed at this time, no matter how the `person` data is modified on Page A, `AppStorageV2` will not respond accordingly. The `person` data on Page B will not be affected.

![PixPin_2024-08-08_00-05-31](readme.assets/PixPin_2024-08-08_00-05-31.gif)

### keys

Return all `keys` in `AppStorageV2`

**Example Code**

```typescript
Button("Return all keys in AppStorageV2").onClick(() => {
  console.log("AppStorageV2.keys()", AppStorageV2.keys());
});
```

## PersistenceV2

`PersistenceV2` is application-level data persistence technology. Data is directly stored on the device disk, and after exiting and re-entering, the data still exists.

The core APIs of `PersistenceV2` are similar to `AppStorageV2`, both providing `connect`, `remove`, and `keys`.

```typescript
import { PersistenceV2 } from '@kit.ArkUI'

@ObservedV2
export class Person {
  @Trace
  age: number = 10
}

@Entry
@ComponentV2
struct Index {
  @Local person: Person = PersistenceV2.connect(Person, () => new Person)!

  build() {
    Column({ space: 10 }) {
      Text("Page A")
      // Preview may not work
      Button(`Age: ${this.person.age}`)
        .onClick(() => {
          this.person.age++
        })
    }
  }
}
```

![PixPin_2024-08-08_00-14-16](readme.assets/PixPin_2024-08-08_00-14-16.gif)

---

At this time, opening the device's file directory, you can see that the data is actually written to the disk.

`/data/app/el2/100/base/com.example.yourpackagename/haps/entry/files/persistent_storage`

![image-20240808003957736](readme.assets/image-20240808003957736.png)

Note that `PersistenceV2` has two additional APIs:

1. `save` - Since changes to non-`@Trace` data won't trigger automatic persistence in `PersistenceV2`, you can manually call `save` for persistence
2. `notifyOnError` - Represents a callback for responding to serialization or deserialization failures, can be used for error handling

### save

Manually persist @Trace data

| save         | Description                                                                                           |
| ------------ | ----------------------------------------------------------------------------------------------------- |
| Parameters   | keyOrType: The key that needs manual persistence; if a type is specified, the key is the type's name. |
| Return Value | None.                                                                                                 |

**Example Code**:

```typescript
import { PersistenceV2 } from '@kit.ArkUI'

export class Person {
  age: number = 10
}

@Entry
@ComponentV2
struct Index {
  @Local person: Person = PersistenceV2.connect(Person, () => new Person)!

  build() {
    Column({ space: 10 }) {
      Text("Page A")
      // Preview may not work
      Button(`Age: ${this.person.age}`)
        .onClick(() => {
          this.person.age++
          console.log("this.person.age", this.person.age)
          PersistenceV2.save(Person)
        })
    }
  }
}
```

---

1. When opening the page for the first time and directly modifying data, you'll find that the UI won't refresh, but the log output shows that age is increasing (because there's no `@ObservedV2` and `@Trace` monitoring)

2. At this time, exit the application and reopen it, you'll find that age becomes the data from the last increase. Because after each modification, the `save` method is called, and the data has been persisted to disk. When the page is reopened, it will re-execute `@Local person: Person = PersistenceV2.connect(Person, () => new Person)!` code, reading data from disk and rendering it to the UI.

### notifyOnError

Represents a callback for responding to serialization or deserialization failures, can be used for error handling

| notifyOnError | Description                                                                                                                 |
| ------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Parameters    | callback: When serialization or deserialization fails, execute this callback; if undefined is passed, cancel this callback. |
| Return Value  | None.                                                                                                                       |

**Example Code**:

```typescript
// Accept serialization failure callbacks
PersistenceV2.notifyOnError((key: string, reason: string, msg: string) => {
  console.error(`error key: ${key}, reason: ${reason}, message: ${msg}`);
});
```

### Serialization Supplement

If the data to be stored is complex nested data, you need to use `@Type` to prevent data loss.

```typescript
import { PersistenceV2, Type } from "@kit.ArkUI";

class Son {
  weight: number = 100;
}

export class Person {
  age: number = 10;
  // Nested complex types, to prevent serialization data failure, need to use @Type to mark the type
  @Type(Son)
  son: Son = new Son();
}
```

## Summary

1. If you need to consider global data sharing, you can consider using `AppStorageV2`
2. If you need data persistence, you can use `PersistenceV2`. Because it directly reads and writes to disk, it's not suitable for persisting large amounts of data, which would cause application performance degradation.
