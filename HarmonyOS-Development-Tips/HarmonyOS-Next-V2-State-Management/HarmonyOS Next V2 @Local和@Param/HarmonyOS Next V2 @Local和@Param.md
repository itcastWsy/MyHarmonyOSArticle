# English Translation Required

This file is marked for translation from: @Local 和@Param.md

Original Chinese file path: 鸿蒙开发技巧\HarmonyOS Next V2 状态管理\HarmonyOS Next V2 @Local 和@Param\@Local 和@Param.md

Please translate the content from the original Chinese file to English.
The translation should maintain:

- Technical accuracy
- Code examples (translate comments but keep code structure)
- Image references
- Link references
- Formatting (headers, lists, etc.)

---

# HarmonyOS Next V2 @Local and @Param

## @Local Background

**@Local** is a state management decorator in the v2 version of **Harmony** application development that corresponds to **@State**. It solves the problem of @State's chaotic detection of state variable changes:

1. State variables decorated with **@State** can be defined internally by the component itself
2. State decorated with **@State** can also be passed from external parent components

This leads to non-unique sources of state data, which can cause problems that are difficult to detect and maintain in large projects. For example, the following code:

```typescript
@Entry
@Component
struct Index {
  @State num: number = 100

  build() {
    Column() {
      Text("Parent component data " + this.num)

      Son({ num: this.num })
      Son()
    }
    .height('100%')
    .width('100%')
  }
}

@Component
struct Son {
  @State num: number = 0

  build() {
    Column() {
      Button(`Child component ${this.num}`)
        .onClick(() => {
          this.num++
        })
    }
  }
}
```

![image-20240718201721853](readme.assets/image-20240718201721853.png)

## @Local Basic Usage

**@Local** was introduced to solve this type of problem:

1. **@Local** can only be used on components decorated with **@ComponentV2**
2. Variables decorated with **@Local** cannot be initialized externally (cannot be passed from parent components), so they must be initialized internally within the component

Let's modify the above code slightly:

```typescript
@Entry
@Component
struct Index {
  @State num: number = 100

  build() {
    Column() {
      Text("Parent component data " + this.num)

      Son({ num: this.num }) // This will cause an error

      Son()
    }
    .height('100%')
    .width('100%')
  }
}

@ComponentV2 // Changed to @ComponentV2 here
struct Son {

  // Changed to @Local here
  @Local num: number = 0

  build() {
    Column() {
      Button(`Child component ${this.num}`)
        .onClick(() => {
          this.num++
        })
    }
  }
}
```

![image-20240718204400711](readme.assets/image-20240718204400711.png)

## @Local vs @State Comparison

|                       | @State                                                                                     | @Local                                                                             |
| --------------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| Parameters            | None.                                                                                      | None.                                                                              |
| Parent initialization | Optional.                                                                                  | External initialization not allowed.                                               |
| Observation ability   | Can observe the variable itself and one layer of member properties, cannot observe deeply. | Can observe the variable itself, deep observation depends on @Trace decorator.     |
| Data transfer         | Can serve as data source and synchronize with state variables in child components.         | Can serve as data source and synchronize with state variables in child components. |

## @Local Special Notes

- @Local supports observing basic types such as number, boolean, string, Object, class, as well as built-in types like Array, Set, Map, Date.
- @Local's observation capability is limited to the **decorated variable itself**. When decorating simple types, it can observe assignments to the variable; when decorating object types, it can only observe assignments to the entire object; when decorating array types, it can observe changes to the entire array and array element items; when decorating built-in types like Array, Set, Map, Date, it can observe changes brought by API calls.

## @Param

**@Param** mainly represents child components receiving data passed from parent components. It can be used together with **@Local**.

## @Param Background

In V1 version state management decorators, the techniques that can be used to handle parent-child parameter passing include:

1. Ordinary properties, no special decorators needed, no one-way synchronization
2. @Prop one-way synchronization, cannot listen to deep-level property changes, also cannot achieve two-way synchronization
3. @Link can achieve two-way synchronization, but cannot listen to deep-level property changes, and cannot be used directly in list rendering technique - ForEach
4. @ObjectLink can achieve two-way synchronization, but must be used with @Observed, and can only be used on custom components

### 1. Ordinary Properties

Ordinary properties, no special decorators needed, no one-way synchronization

```typescript
@Entry
@Component
struct Index {
  @State num: number = 100

  build() {
    Column() {
      // Data passed from parent component
      Son({ num: this.num })
        .onClick(() => {
          this.num++
          console.log("", this.num)
        })
    }
    .height('100%')
    .width('100%')
  }
}

@Component
struct Son {
  num: number = 0

  build() {
    Column() {
      Button(`Child component ${this.num}`)
    }
  }
}
```

![image-20240718215338082](readme.assets/image-20240718215338082.png)

### 2. @Prop One-way Synchronization

@Prop one-way synchronization

1. Cannot listen to deep-level property changes
2. Also cannot achieve **two-way synchronization**

Based on the above code, adding **@Prop** can detect updates to basic type data

```typescript
@Component
struct Son {
  @Prop num: number = 0
```

But it cannot detect deep-level property changes, such as:

```typescript
class Animal {
  dog: Dog = {
    age: 100
  }
}

class Dog {
  age: number = 100
}

@Entry
@Component
struct Index {
  @State
  animal: Animal = new Animal()

  build() {
    Column() {
      // Data passed from parent component
      Son({ dog: this.animal.dog })
        .onClick(() => {
          this.animal.dog.age++
          console.log("", this.animal.dog.age)
        })
    }
    .height('100%')
    .width('100%')
  }
}

@Component
struct Son {
  @Prop dog: Dog

  build() {
    Column() {
      Button(`Child component ${this.dog.age}`)
    }
  }
}
```

![image-20240718215929872](readme.assets/image-20240718215929872.png)

### 3. @Link Two-way Synchronization

> @Link usage is basically the same as @Prop

Can achieve two-way synchronization, but:

1. Cannot listen to deep-level property changes

2. Cannot be used directly in list rendering technique - **ForEach** _The following code demonstrates this effect_

   ```typescript
   class Dog {
     age: number = 100
   }

   @Entry
   @Component
   struct Index {
     @State
     dogList: Dog [] = [new Dog(), new Dog(), new Dog(), new Dog()]

     build() {
       Column() {
         ForEach(this.dogList, (item: Dog) => {
           // This will cause an error: Assigning the attribute 'item' to the '@Link' decorated attribute 'dog' is not allowed. <ArkTSCheck>
           Son({ dog: item })
             .onClick(() => {
               item.age++
               console.log("", item.age)
             })
         })

       }
       .height('100%')
       .width('100%')
     }
   }

   @Component
   struct Son {
     @Link dog: Dog

     build() {
       Column() {
         Button(`Child component ${this.dog.age}`)
       }
     }
   }
   ```

   ***

   ![image-20240718220523209](readme.assets/image-20240718220523209.png)

### 4. @ObjectLink

@ObjectLink can achieve two-way synchronization, but must be used with @Observed, and can only be used on **custom components**

![image-20240715182615579](readme.assets/image-20240715182615579.png)

### Summary

As you can see, using the V1 version of parent-child parameter passing techniques is very complex and difficult to use directly.

## @Param Introduction

Param represents state passed into components from external sources, enabling data synchronization between parent and child components:

- Variables decorated with @Param support local initialization, but direct modification of the variable itself within the component is not allowed.
  - If not locally initialized, then **@Require** must be added
- @Param can achieve one-way synchronization
- @Param can detect deep-level property modifications, but such modifications must be updates to the entire object at the data source
- If @Param also wants to deeply listen to modifications of individual properties, then **@ObservedV2** and **@Trace** need to be used

The following code mainly demonstrates: **@Param can detect deep-level property modifications, but such modifications must be updates to the entire object at the data source**

```typescript
class Person {
  age: number = 100
}
@Entry
@ComponentV2
struct Index {
  @Local
  person: Person = new Person()
  build() {
    Column() {
      Son({ age: this.person.age })
        .onClick(() => {
          this.person.age++
          console.log("", this.person.age)
          if (this.person.age === 105) {
            const p = new Person()
            p.age = 200
            // Overall update, child component can detect it
            this.person = p
          }
        })
    }
    .height('100%')
    .width('100%')
  }
}

@ComponentV2
struct Son {
  // Either set @Require to indicate parent component must pass data
  // Or set default value
  @Require @Param age: number
  build() {
    Column() {
      Button(`Child component ${this.age}`)
    }
  }
}
```

![image-20240718222428155](readme.assets/image-20240718222428155.png)

## Summary

1. @Local can be seen as a replacement for @State, specifically representing internal state of components
2. @Param can be seen as a replacement for @Prop @Link @ObjectLink, more rigorous
3. @Local and @Param used together can only be used on custom components decorated with @ComponentV2
