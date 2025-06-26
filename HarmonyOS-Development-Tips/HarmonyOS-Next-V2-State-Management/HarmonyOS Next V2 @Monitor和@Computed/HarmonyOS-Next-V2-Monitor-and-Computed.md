# English Translation Required

This file is marked for translation from: @Monitor 和@Computed.md

Original Chinese file path: 鸿蒙开发技巧\HarmonyOS Next V2 状态管理\HarmonyOS Next V2 @Monitor 和@Computed\@Monitor 和@Computed.md

Please translate the content from the original Chinese file to English.
The translation should maintain:

- Technical accuracy
- Code examples (translate comments but keep code structure)
- Image references
- Link references
- Formatting (headers, lists, etc.)

---

# HarmonyOS Next V2 @Monitor and @Computed

## @Monitor Introduction

`@Monitor` is a technology used to monitor state variable modifications in the V2 version of state management.

It can be directly used in:

1. Custom components decorated with `@ComponentV2`, for state variables modified by `@Local`, `@Param`, `@Provider`, `@Consumer`, `@Computed`
2. For deep-level data, such as deep objects, object arrays, etc., it needs to be used together with `@ObservedV2` and `@Trace`
3. Can monitor multiple properties simultaneously
4. Can obtain data changes before and after the monitored property is modified

## Comparison with @Watch in State Management V1

`@Monitor` is much more powerful than `@Watch`:

1. `@Watch` cannot be used with components decorated by `@ComponentV2`
2. `@Watch` doesn't have deep monitoring capabilities
3. `@Watch` cannot monitor multiple properties simultaneously
4. `@Watch` cannot detect changes before and after property modification

## @Monitor Monitoring Single Property

```typescript
@Entry
@ComponentV2
struct Index {
  @Local num: number = 100

  @Monitor("num")
  changeNum() {
    console.log("Data modification detected!")
  }

  build() {
    Column() {

      Button(`Click to modify ${this.num}`)
        .onClick(() => {
          this.num++
        })
    }
    .width("100%")
    .height("100%")
  }
}
```

## @Monitor Monitoring Multiple Properties Simultaneously

```typescript
@Entry
@ComponentV2
struct Index {
  @Local num: number = 100
  @Local age: number = 200

  // Monitor multiple state modifications simultaneously
  @Monitor("num","age")
  changeNum() {
    console.log("Data modification detected!")
  }

  build() {
    Column() {
      Button(`Click to modify num ${this.num}`)
        .onClick(() => {
          this.num++
        })
      Button(`Click to modify age ${this.age}`)
        .onClick(() => {
          this.age++
        })
    }
    .width("100%")
    .height("100%")
  }
}
```

## @Monitor Callback Function

The `@Monitor` callback function can provide us with these capabilities:

1. If multiple states are monitored and only one state changes, we can know which specific state has changed
2. When a state changes, we can get both the before and after values

The parameter of @Monitor's callback function is [IMonitor](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-new-monitor-V5#imonitor%E7%B1%BB%E5%9E%8B), which is an object with two properties:

1. `dirty` - a string array containing the names of modified states
2. `value` - a function that returns a new object containing `path: name of modified state, before: data before modification, now: data after modification`. When calling `value()`, if no parameters are passed and you are modifying multiple states simultaneously, it will only return the first state. If a parameter (state variable) is passed, it will return information about that state variable.

```typescript
@Entry
@ComponentV2
struct Index {
  @Local num: number = 100
  @Local age: number = 200

  // Monitor multiple state modifications simultaneously
  @Monitor("num","age")
  changeNum(Monitor: IMonitor) {
    console.log("Modified states", Monitor.dirty)
    console.log("Monitor.value()", JSON.stringify(Monitor.value("age")))
  }

  build() {
    Column() {
      Button(`Modify both num and age ${this.num}  ${this.age}`)
        .onClick(() => {
          this.num++
          this.age++
        })

    }
    .width("100%")
    .height("100%")
  }
}
```

## @Monitor Deep Monitoring

`@Monitor` needs to be used together with `@ObservedV2` and `@Trace` to achieve deep monitoring effects. Note that:

1. `@Monitor` can be written directly in classes decorated with `@ObservedV2`
2. `@Monitor` can also be written in normal components

```typescript
@ObservedV2
class Person {
  @Trace son: Son = new Son()
}

@ObservedV2
class Son {
  // @Monitor can be written directly in classes decorated with @ObservedV2
  @Monitor("weight")
  weightChange() {
    console.log("1 Son's weight has been modified")
  }

  @Trace weight: number = 200
}

@Entry
@ComponentV2
struct Index {
  person: Person = new Person()
  // @Monitor can also be written in normal components
  @Monitor("person.son.weight")
  weightChange() {
    console.log("2 Son's weight has been modified")
  }

  build() {
    Column() {
      Button(`Modify son's weight ${this.person.son.weight}`)
        .onClick(() => {
          this.person.son.weight++
        })
    }
    .width("100%")
    .height("100%")
  }
}
```

## @Monitor Limitations

In actual development, `@Monitor` has some limitations. It cannot monitor changes caused by API calls on built-in types (`Array`, `Map`, `Date`, `Set`). For example, when you're monitoring an entire array and use common methods like `push`, `splice` to modify the array, it cannot be detected. However, when the entire array is reassigned, its change can be detected.

```typescript
@ObservedV2
class Person {
  @Trace name: string = "XiaoMing"
}

@Entry
@ComponentV2
struct Index {
  @Local
  personList: Person[] = [new Person()]

  @Monitor("personList")
  weightChange() {
    console.log("Array modification detected")
  }

  build() {
    Column() {
      Button("Add one")
        .onClick(() => {
          // 1 Invalid - Cannot detect array modification
          this.personList.push(new Person())

          // 2 Valid - Array modification detected
          // const newPerson = [...this.personList, new Person()]
          // this.personList = newPerson
        })
      ForEach(this.personList, (item: Person) => {
        Text(item.name)
      })
    }
    .width("100%")
    .height("100%")
  }
}
```

Additionally, you can indirectly detect array element changes through dot syntax or by monitoring array length.

**Dot Syntax**

```typescript
@ObservedV2
class Person {
  @Trace name: string = "XiaoMing"
}

@Entry
@ComponentV2
struct Index {
  @Local
  personList: Person[] = [new Person()]

  @Monitor("personList.0")
  // If you want to monitor a specific property in an object: @Monitor("personList.0.name")
  weightChange() {
    console.log("Array modification detected")
  }

  build() {
    Column() {
      Button("Add one")
        .onClick(() => {
          const p = new Person()
          p.name = "XiaoHei"
          this.personList[0] = p
        })
      ForEach(this.personList, (item: Person) => {
        Text(item.name)
      })
    }
    .width("100%")
    .height("100%")
  }
}
```

**Monitoring Array Length Changes**

```typescript
@ObservedV2
class Person {
  @Trace name: string = "XiaoMing"
}

@Entry
@ComponentV2
struct Index {
  @Local
  personList: Person[] = [new Person()]

  @Monitor("personList.length")
  weightChange() {
    console.log("Array modification detected")
  }

  build() {
    Column() {
      Button("Add one")
        .onClick(() => {
          const p = new Person()
          p.name = "XiaoHei"
          this.personList.push(p)
        })
      ForEach(this.personList, (item: Person) => {
        Text(item.name)
      })
    }
    .width("100%")
    .height("100%")
  }
}
```

## @Computed

`@Computed` is a computed property that can monitor data changes and calculate new values accordingly. Usage is quite simple:

```typescript
@Entry
@ComponentV2
struct Index {
  @Local num: number = 100

  @Computed
  get numText() {
    return this.num * 2
  }

  build() {
    Column() {
      Button("Modify")
        .onClick(() => {
          this.num++
        })
      Text(`Original data ${this.num}`)
      Text(`After computation ${this.numText}`)
    }
    .width("100%")
    .height("100%")
  }
}
```
